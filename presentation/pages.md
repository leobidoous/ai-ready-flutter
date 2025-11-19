# Presentation Pages - Clean Architecture

## 📚 Visão Geral

As **Pages** na camada de **Presentation** são responsáveis por **compor a interface do usuário** e **orquestrar a interação** entre widgets e controllers. Elas seguem padrões específicos para diferentes tipos de operações: listagem, formulários, detalhes e operações especializadas.

### 🎯 Princípios Fundamentais das Pages

**O QUE as Pages FAZEM:**
- ✅ **Compõem Interface**: Estruturam e organizam widgets na tela
- ✅ **Orquestram Controllers**: Coordenam múltiplos controllers quando necessário
- ✅ **Gerenciam Navegação**: Controlam fluxos de navegação e estado local
- ✅ **Tratam Estados da UI**: Loading, error, success, empty states
- ✅ **Implementam UX Patterns**: Pull-to-refresh, infinite scroll, multi-step forms

**O QUE as Pages NÃO FAZEM:**
- ❌ **Não contêm regras de negócio**: Apenas consomem dados dos controllers
- ❌ **Não fazem comunicação direta**: Usam controllers para acessar dados
- ❌ **Não implementam transformação**: Recebem dados prontos para UI
- ❌ **Não gerenciam estado de negócio**: Apenas estado local da UI
- ❌ **Não contêm validações de domínio**: Apenas validações de UI/UX

### 🏗️ Localização e Estrutura

```
lib/src/presentation/pages/
├── enrollment_list/
│   ├── enrollments_page.dart           # Lista principal
│   └── widgets/
│       ├── enrollment_list_item.dart   # Item da lista
│       ├── enrollment_filters.dart     # Filtros
│       └── enrollment_shimmer.dart     # Loading state
├── enrollment_details/
│   ├── enrollment_details_page.dart    # Detalhes com tabs
│   └── widgets/
│       ├── enrollment_details_view.dart
│       └── enrollment_simulation_view.dart
├── create_enrollment/
│   ├── create_enrollment_page.dart     # Form multi-step
│   └── widgets/
│       ├── personal_data_form_view.dart
│       ├── payment_options_view.dart
│       └── resume_view.dart
└── feature_page/
    ├── feature_page.dart               # Page principal
    └── widgets/                        # Widgets específicos
```

---

## 🔍 Anatomia de uma Page

### Estrutura Base

```dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

class FeaturePage extends StatefulWidget {
  const FeaturePage({super.key, this.args});

  final FeaturePageArgs? args;

  @override
  State<FeaturePage> createState() => _FeaturePageState();
}

class _FeaturePageState extends State<FeaturePage> {
  final _controller = DM.i.get<FeatureController>();
  
  @override
  void initState() {
    super.initState();
    _initializeData();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _initializeData() {
    // Carregamento inicial de dados
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Feature'),
      body: CustomListenableBuilder<FeatureState, IFeatureFailure, FeatureState>(
        controller: _controller,
        builder: (context, state, isLoading, hasError, error) {
          // Tratamento de estados
          return _buildContent(state, isLoading, hasError, error);
        },
      ),
    );
  }
}
```

### 🔑 Elementos Essenciais

1. **Args Class**: Tipificação de parâmetros da página
2. **Controller Injection**: Dependency injection via DM.i.get
3. **CustomListenableBuilder**: Binding reativo com controller
4. **State Management**: initState/dispose lifecycle
5. **Error/Loading Handling**: Tratamento completo de estados

---

## 📚 Exemplos Práticos Reais

### 1. EnrollmentsPage - Lista com Filtros e Paginação

```dart
class EnrollmentsPage extends StatefulWidget {
  const EnrollmentsPage({super.key});

  @override
  State<EnrollmentsPage> createState() => _EnrollmentsPageState();
}

class _EnrollmentsPageState extends State<EnrollmentsPage> {
  late final PagedListController<IEnrollmentFailure, EnrollmentEntity> _listController;
  final _controller = DM.i.get<EnrollmentsController>();
  final _expansionKey = GlobalKey<CustomExpansionState>();
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _setupScrollListener();
    _setupPagedList();
  }

  void _setupScrollListener() {
    double scrollOffset = 0;
    _scrollController.addListener(() {
      final offset = _scrollController.offset;
      if (offset <= scrollOffset || offset <= 0) {
        _expansionKey.currentState?.reverse();
      } else {
        _expansionKey.currentState?.forward();
      }
      scrollOffset = _scrollController.offset;
    });
  }

  void _setupPagedList() {
    _listController = PagedListController(
      pageSize: _controller.filters.pagination.limit ?? 30,
      firstPageKey: 1,
    );
    
    _listController.setListener(({required int pageKey, int? pageSize}) async {
      _controller.filters = _controller.filters.copyWith(
        pagination: _controller.filters.pagination.copyWith(pageNumber: pageKey),
      );
      
      return _controller.fetchEnrollments(_controller.filters).then((onValue) {
        if (_controller.hasError) throw _controller.error!;
        return _controller.state;
      });
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    _listController.dispose();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Inscrições'),
      body: Column(
        children: [
          // Filtros expansíveis
          _buildFiltersSection(),
          
          // Lista paginada
          Expanded(
            child: PagedListView(
              listController: _listController,
              scrollController: _scrollController,
              padding: EdgeInsets.symmetric(
                horizontal: Spacing.sm.value,
                vertical: Spacing.md.value,
              ),
              separatorBuilder: (_, index) => Spacing.sm.vertical,
              itemBuilder: (_, item, index) => _EnrollmentListItem(item: item),
              noItemsFoundIndicatorBuilder: (_, onRefresh) => _buildEmptyState(onRefresh),
              newPageErrorIndicatorBuilder: (_, error, onRefresh) => _buildError(error, onRefresh),
              firstPageProgressIndicatorBuilder: (_) => _EnrollmentListShimmer(),
              firstPageErrorIndicatorBuilder: (_, error, onRefresh) => _buildError(error, onRefresh),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildFiltersSection() {
    return DecoratedBox(
      decoration: BoxDecoration(
        boxShadow: [context.theme.shadowLightmodeLevel1],
        color: context.theme.scaffoldBackgroundColor,
      ),
      child: CustomExpansion(
        key: _expansionKey,
        allowExpand: false,
        showTrailing: false,
        allowDismissOnBody: false,
        initialState: ExpansionState.opened,
        body: CustomListenableBuilder(
          controller: _controller,
          builder: (context, state, isLoading, hasError, error) {
            return _EnrollmentsFilters(
              filters: _controller.filters,
              isEnabled: !isLoading,
              onChanged: (filters) {
                setState(() {
                  _controller.filters = filters;
                  _listController.refresh();
                });
              },
            );
          },
        ),
      ),
    );
  }
}
```

### 2. EnrollmentDetailsPage - Detalhes com Tabs

```dart
class EnrollmentDetailsPageArgs {
  EnrollmentDetailsPageArgs({this.idOrigin = '', this.businessKey = ''});

  final String idOrigin;
  final String businessKey;
}

class EnrollmentDetailsPage extends StatefulWidget {
  const EnrollmentDetailsPage({super.key, required this.args});

  final EnrollmentDetailsPageArgs args;

  @override
  State<EnrollmentDetailsPage> createState() => _EnrollmentDetailsPageState();
}

class _EnrollmentDetailsPageState extends State<EnrollmentDetailsPage>
    with SingleTickerProviderStateMixin {
  final _enrollmentDetailsController = DM.i.get<EnrollmentDetailsController>();
  final _companiesController = DM.i.get<CompaniesController>();
  late final TabController _tabController;
  int _tabIndex = 0;

  @override
  void initState() {
    super.initState();
    _initializeData();
    _setupTabController();
  }

  void _initializeData() {
    _getEnrollmentDetails();
    _companiesController.fetchCompanies(term: '');
  }

  void _setupTabController() {
    _tabController = TabController(length: 2, vsync: this);
    _tabController.addListener(() {
      setState(() {
        _tabIndex = _tabController.index;
      });
    });
  }

  @override
  void dispose() {
    _enrollmentDetailsController.dispose();
    _companiesController.dispose();
    _tabController.dispose();
    super.dispose();
  }

  void _getEnrollmentDetails() {
    _enrollmentDetailsController.getEnrollmentDetails(
      idOrigin: widget.args.idOrigin,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Jornada de matrícula', enableShadow: false),
      body: Column(
        children: [
          // Tab Bar
          _buildTabBar(),
          
          // Tab Content
          Expanded(
            child: CustomListenableBuilder(
              controller: _enrollmentDetailsController,
              builder: (context, enrollment, isLoading, hasError, error) {
                if (isLoading) {
                  return Center(child: CustomLoading());
                } else if (hasError) {
                  return _buildErrorState(error);
                }
                
                return TabBarView(
                  controller: _tabController,
                  children: [
                    _buildDetailsTab(enrollment),
                    _buildSimulationTab(),
                  ],
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildTabBar() {
    return DecoratedBox(
      decoration: BoxDecoration(
        color: context.colorScheme.surface,
        boxShadow: [context.theme.shadowLightmodeLevel0],
        border: Border(
          bottom: BorderSide(width: .1, color: context.theme.cardColor),
        ),
      ),
      child: TabBar(
        controller: _tabController,
        labelPadding: EdgeInsets.symmetric(
          horizontal: Spacing.sm.value,
          vertical: Spacing.xs.value,
        ),
        tabs: [
          _tabTitle('Detalhes', isSelected: _tabIndex == 0),
          _tabTitle('Simulação', isSelected: _tabIndex == 1),
        ],
      ),
    );
  }

  Widget _buildDetailsTab(EnrollmentEntity enrollment) {
    return CustomScrollContent(
      expanded: true,
      alwaysScrollable: true,
      onRefresh: _getEnrollmentDetails,
      padding: EdgeInsets.symmetric(
        vertical: Spacing.md.value,
        horizontal: Spacing.sm.value,
      ),
      child: SafeArea(
        child: _EnrollmentDetailsView(enrollment: enrollment),
      ),
    );
  }
}
```

### 3. CreateEnrollmentPage - Form Multi-Step

```dart
class CreateEnrollmentPageArgs {
  CreateEnrollmentPageArgs({
    this.cpf = '',
    required this.offer,
    required this.isEasyPay,
    required this.shiftType,
    required this.offerPricing,
    this.daysSelected = const [],
    required this.easyPayAvailable,
  });

  final String cpf;
  final bool isEasyPay;
  final OfferEntity offer;
  final bool easyPayAvailable;
  final OfferShiftType shiftType;
  final OfferPricingEntity offerPricing;
  final List<OfferWeekDay> daysSelected;
}

class CreateEnrollmentPage extends StatefulWidget {
  const CreateEnrollmentPage({
    super.key,
    required this.args,
    required this.goToEnrollmentDetails,
  });

  final CreateEnrollmentPageArgs args;
  final Function(String idOrigin)? goToEnrollmentDetails;

  @override
  State<CreateEnrollmentPage> createState() => _CreateEnrollmentPageState();
}

class _CreateEnrollmentPageState extends State<CreateEnrollmentPage> {
  final _controller = DM.i.get<EnrollmentController>();
  late final PageController _pageController;
  int _nSteps = 3;
  int _pageIndex = 0;

  @override
  void initState() {
    super.initState();
    _initializeForm();
    _setupPageController();
  }

  void _initializeForm() {
    final offerPrice = widget.args.isEasyPay 
        ? widget.args.offerPricing.easyPay 
        : widget.args.offerPricing.base;

    // Calcula número de steps dinamicamente
    if (offerPrice.paymentMethods.isNotEmpty) _nSteps++;
    if (offerPrice.hasVoucherSupport) _nSteps++;

    // Inicializa dados do controller
    _controller.update(
      _controller.state.copyWith(
        personalData: _controller.state.personalData.copyWith(
          cpfPermission: _controller.state.personalData.cpfPermission.copyWith(
            cpf: widget.args.cpf,
          ),
        ),
        enrollment: _controller.state.enrollment.copyWith(
          isEasyPay: widget.args.isEasyPay,
          offerPricing: widget.args.offerPricing,
          source: widget.args.offer.source,
          // ... outros campos
        ),
      ),
    );
  }

  void _setupPageController() {
    _pageController = PageController(initialPage: _pageIndex);
  }

  @override
  void dispose() {
    _pageController.dispose();
    _controller.dispose();
    super.dispose();
  }

  void _onBackTap() {
    if (_pageIndex > 0) {
      _pageController.previousPage(
        duration: Durations.long2,
        curve: Curves.decelerate,
      );
    } else {
      _onConfirmExitBSheet();
    }
  }

  void _onNextPage() {
    _pageController.nextPage(
      duration: Durations.long2,
      curve: Curves.decelerate,
    );
  }

  @override
  Widget build(BuildContext context) {
    return PopScope(
      canPop: false,
      onPopInvokedWithResult: (didPop, result) => !didPop ? _onBackTap() : null,
      child: Scaffold(
        appBar: _buildAppBar(),
        body: CustomListenableBuilder(
          controller: _controller,
          builder: (context, state, isLoading, hasError, error) {
            return PageView(
              physics: const NeverScrollableScrollPhysics(),
              onPageChanged: (index) => setState(() => _pageIndex = index),
              controller: _pageController,
              children: _buildFormSteps(state),
            );
          },
        ),
      ),
    );
  }

  List<Widget> _buildFormSteps(CreateEnrollmentEntity state) {
    final steps = <Widget>[
      _PersonalDataFormView(
        personalData: state.personalData,
        onNext: (personalData, offerPricing, isEasyPay) => _onNextPage(),
        onBack: _onBackTap,
      ),
    ];

    // Adiciona steps condicionalmente
    if (_offerPrice.paymentMethods.isNotEmpty) {
      steps.add(_PaymentOptionsView(/* ... */));
    }

    if (_offerPrice.hasVoucherSupport) {
      steps.add(_VoucherView(/* ... */));
    }

    steps.addAll([
      _AddressFormView(/* ... */),
      _ResumeView(/* ... */),
      _RequestStatusView(
        goToEnrollmentDetails: widget.goToEnrollmentDetails,
      ),
    ]);

    return steps;
  }

  PreferredSizeWidget? _buildAppBar() {
    if ((_pageIndex + 1) / _nSteps > 1) return null;

    return CustomAppBar(
      titleWidget: AutoSizeText(
        'Dados do candidato',
        maxLines: 1,
        style: context.textTheme.titleMedium,
      ),
      onBackTap: _onConfirmExitBSheet,
      progress: (_pageIndex + 1) / _nSteps,
      actions: [ShareFormButton()],
    );
  }
}
```

---

## 🎨 Padrões de Implementação

### 1. Lista Simples com Pull-to-Refresh

```dart
class SimpleListPage extends StatefulWidget {
  const SimpleListPage({super.key});

  @override
  State<SimpleListPage> createState() => _SimpleListPageState();
}

class _SimpleListPageState extends State<SimpleListPage> {
  final _controller = DM.i.get<ItemListController>();

  @override
  void initState() {
    super.initState();
    _controller.loadItems();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Items'),
      body: CustomListenableBuilder<ItemListState, IItemFailure, ItemListState>(
        controller: _controller,
        builder: (context, state, isLoading, hasError, error) {
          if (isLoading && state.items.isEmpty) {
            return const Center(child: CustomLoading());
          }

          if (hasError && state.items.isEmpty) {
            return CognaRequestError(
              error: error!,
              onRetry: () => _controller.loadItems(),
            );
          }

          if (state.items.isEmpty) {
            return const EmptyStateWidget(
              message: 'Nenhum item encontrado',
            );
          }

          return RefreshIndicator(
            onRefresh: () => _controller.refreshItems(),
            child: ListView.builder(
              itemCount: state.items.length,
              itemBuilder: (context, index) => ItemTile(
                item: state.items[index],
                onTap: () => _openItemDetails(state.items[index]),
              ),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _createNewItem(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### 2. Form Simples com Validação

```dart
class CreateItemPage extends StatefulWidget {
  const CreateItemPage({super.key, this.itemToEdit});

  final ItemEntity? itemToEdit;

  @override
  State<CreateItemPage> createState() => _CreateItemPageState();
}

class _CreateItemPageState extends State<CreateItemPage> {
  final _controller = DM.i.get<CreateItemController>();
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _nameController;
  late final TextEditingController _descriptionController;

  @override
  void initState() {
    super.initState();
    _initializeControllers();
    _loadInitialData();
  }

  void _initializeControllers() {
    _nameController = TextEditingController(text: widget.itemToEdit?.name ?? '');
    _descriptionController = TextEditingController(text: widget.itemToEdit?.description ?? '');
  }

  void _loadInitialData() {
    if (widget.itemToEdit != null) {
      _controller.loadItem(widget.itemToEdit!);
    }
  }

  @override
  void dispose() {
    _nameController.dispose();
    _descriptionController.dispose();
    _controller.dispose();
    super.dispose();
  }

  void _onSave() {
    if (_formKey.currentState?.validate() == true) {
      final item = ItemEntity(
        id: widget.itemToEdit?.id ?? '',
        name: _nameController.text,
        description: _descriptionController.text,
      );

      _controller.saveItem(item).then((result) {
        result.fold(
          (error) => _showError(error),
          (success) => Nav.to.pop(response: success),
        );
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(
        title: widget.itemToEdit == null ? 'Criar Item' : 'Editar Item',
        actions: [
          CustomListenableBuilder(
            controller: _controller,
            builder: (context, state, isLoading, hasError, error) {
              return TextButton(
                onPressed: isLoading ? null : _onSave,
                child: isLoading 
                    ? const SizedBox.square(
                        dimension: 16,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Text('Salvar'),
              );
            },
          ),
        ],
      ),
      body: Form(
        key: _formKey,
        child: CustomScrollContent(
          padding: EdgeInsets.all(Spacing.md.value),
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(labelText: 'Nome'),
                validator: (value) => value?.isEmpty == true ? 'Campo obrigatório' : null,
                textCapitalization: TextCapitalization.words,
              ),
              Spacing.md.vertical,
              TextFormField(
                controller: _descriptionController,
                decoration: const InputDecoration(labelText: 'Descrição'),
                maxLines: 3,
                textCapitalization: TextCapitalization.sentences,
              ),
              
              // Error handling
              CustomListenableBuilder(
                controller: _controller,
                builder: (context, state, isLoading, hasError, error) {
                  if (hasError) {
                    return Padding(
                      padding: EdgeInsets.only(top: Spacing.md.value),
                      child: ErrorBanner(
                        error: error!,
                        onDismiss: _controller.clearError,
                      ),
                    );
                  }
                  return const SizedBox.shrink();
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 3. Detalhes com Múltiplas Seções

```dart
class ItemDetailsPage extends StatefulWidget {
  const ItemDetailsPage({super.key, required this.itemId});

  final String itemId;

  @override
  State<ItemDetailsPage> createState() => _ItemDetailsPageState();
}

class _ItemDetailsPageState extends State<ItemDetailsPage> {
  final _controller = DM.i.get<ItemDetailsController>();
  final _commentsController = DM.i.get<ItemCommentsController>();

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  void _loadData() {
    _controller.loadItem(widget.itemId);
    _commentsController.loadComments(widget.itemId);
  }

  @override
  void dispose() {
    _controller.dispose();
    _commentsController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // App Bar expansível
          CustomListenableBuilder(
            controller: _controller,
            builder: (context, item, isLoading, hasError, error) {
              return SliverAppBar(
                expandedHeight: 200,
                pinned: true,
                flexibleSpace: FlexibleSpaceBar(
                  title: Text(item.name),
                  background: item.imageUrl.isNotEmpty
                      ? CachedNetworkImage(imageUrl: item.imageUrl)
                      : const ColoredBox(color: Colors.grey),
                ),
              );
            },
          ),

          // Conteúdo principal
          SliverToBoxAdapter(
            child: CustomListenableBuilder(
              controller: _controller,
              builder: (context, item, isLoading, hasError, error) {
                if (isLoading) {
                  return const ItemDetailsShimmer();
                }

                if (hasError) {
                  return CognaRequestError(
                    error: error!,
                    onRetry: () => _controller.loadItem(widget.itemId),
                  );
                }

                return Column(
                  children: [
                    _ItemInfoSection(item: item),
                    _ItemSpecsSection(item: item),
                    _ItemActionsSection(
                      item: item,
                      onEdit: () => _editItem(item),
                      onDelete: () => _deleteItem(item),
                    ),
                  ],
                );
              },
            ),
          ),

          // Comentários
          SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(Spacing.md.value),
              child: Text(
                'Comentários',
                style: context.textTheme.titleLarge,
              ),
            ),
          ),

          CustomListenableBuilder(
            controller: _commentsController,
            builder: (context, commentsState, isLoading, hasError, error) {
              if (isLoading) {
                return const SliverToBoxAdapter(
                  child: CommentsShimmer(),
                );
              }

              if (hasError) {
                return SliverToBoxAdapter(
                  child: CognaRequestError(
                    error: error!,
                    onRetry: () => _commentsController.loadComments(widget.itemId),
                  ),
                );
              }

              return SliverList.builder(
                itemCount: commentsState.comments.length,
                itemBuilder: (context, index) => CommentTile(
                  comment: commentsState.comments[index],
                ),
              );
            },
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _addComment(),
        child: const Icon(Icons.add_comment),
      ),
    );
  }
}
```

### 4. Dashboard com Múltiplos Controllers

```dart
class DashboardPage extends StatefulWidget {
  const DashboardPage({super.key});

  @override
  State<DashboardPage> createState() => _DashboardPageState();
}

class _DashboardPageState extends State<DashboardPage> {
  final _statsController = DM.i.get<DashboardStatsController>();
  final _recentItemsController = DM.i.get<RecentItemsController>();
  final _notificationsController = DM.i.get<NotificationsController>();

  @override
  void initState() {
    super.initState();
    _loadAllData();
  }

  void _loadAllData() {
    _statsController.loadStats();
    _recentItemsController.loadRecentItems();
    _notificationsController.loadNotifications();
  }

  @override
  void dispose() {
    _statsController.dispose();
    _recentItemsController.dispose();
    _notificationsController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(
        title: 'Dashboard',
        actions: [
          IconButton(
            onPressed: _loadAllData,
            icon: const Icon(Icons.refresh),
          ),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: () async => _loadAllData(),
        child: CustomScrollView(
          slivers: [
            // Stats Section
            SliverToBoxAdapter(
              child: CustomListenableBuilder(
                controller: _statsController,
                builder: (context, stats, isLoading, hasError, error) {
                  if (isLoading) {
                    return const StatsShimmer();
                  }

                  if (hasError) {
                    return ErrorBanner(
                      error: error!,
                      onRetry: () => _statsController.loadStats(),
                    );
                  }

                  return StatsSection(stats: stats);
                },
              ),
            ),

            // Recent Items
            SliverToBoxAdapter(
              child: Padding(
                padding: EdgeInsets.all(Spacing.md.value),
                child: Text(
                  'Itens Recentes',
                  style: context.textTheme.titleLarge,
                ),
              ),
            ),

            CustomListenableBuilder(
              controller: _recentItemsController,
              builder: (context, recentItems, isLoading, hasError, error) {
                if (isLoading) {
                  return const SliverToBoxAdapter(
                    child: RecentItemsShimmer(),
                  );
                }

                if (hasError) {
                  return SliverToBoxAdapter(
                    child: ErrorBanner(
                      error: error!,
                      onRetry: () => _recentItemsController.loadRecentItems(),
                    ),
                  );
                }

                return SliverList.builder(
                  itemCount: recentItems.items.length,
                  itemBuilder: (context, index) => RecentItemTile(
                    item: recentItems.items[index],
                    onTap: () => _openItemDetails(recentItems.items[index]),
                  ),
                );
              },
            ),

            // Notifications
            SliverToBoxAdapter(
              child: Padding(
                padding: EdgeInsets.all(Spacing.md.value),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      'Notificações',
                      style: context.textTheme.titleLarge,
                    ),
                    TextButton(
                      onPressed: () => _openAllNotifications(),
                      child: const Text('Ver todas'),
                    ),
                  ],
                ),
              ),
            ),

            CustomListenableBuilder(
              controller: _notificationsController,
              builder: (context, notifications, isLoading, hasError, error) {
                if (isLoading) {
                  return const SliverToBoxAdapter(
                    child: NotificationsShimmer(),
                  );
                }

                if (hasError) {
                  return SliverToBoxAdapter(
                    child: ErrorBanner(
                      error: error!,
                      onRetry: () => _notificationsController.loadNotifications(),
                    ),
                  );
                }

                return SliverList.builder(
                  itemCount: math.min(notifications.items.length, 5), // Máximo 5
                  itemBuilder: (context, index) => NotificationTile(
                    notification: notifications.items[index],
                    onTap: () => _handleNotification(notifications.items[index]),
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 📋 Template para Pages

### Args Class

```dart
class [Feature]PageArgs {
  [Feature]PageArgs({
    required this.requiredParam,
    this.optionalParam,
  });

  final String requiredParam;
  final String? optionalParam;
}
```

### Estrutura Principal

```dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

/// Page que exibe [descrição da funcionalidade]
/// 
/// Responsável por [lista de responsabilidades]
/// e permite [lista de ações do usuário].
class [Feature]Page extends StatefulWidget {
  const [Feature]Page({super.key, this.args});

  final [Feature]PageArgs? args;

  @override
  State<[Feature]Page> createState() => _[Feature]PageState();
}

class _[Feature]PageState extends State<[Feature]Page> {
  final _controller = DM.i.get<[Feature]Controller>();
  // outros controllers, se necessário
  
  // Controllers de UI (ScrollController, TabController, etc.)
  late final ScrollController _scrollController;

  @override
  void initState() {
    super.initState();
    _initializeData();
    _setupUIControllers();
  }

  void _initializeData() {
    // Carregamento inicial baseado nos args
    if (widget.args?.requiredParam != null) {
      _controller.loadData(widget.args!.requiredParam);
    }
  }

  void _setupUIControllers() {
    _scrollController = ScrollController();
    // Setup de outros controllers de UI
  }

  @override
  void dispose() {
    // Dispose de controllers de negócio
    _controller.dispose();
    
    // Dispose de controllers de UI
    _scrollController.dispose();
    
    super.dispose();
  }

  // Métodos de ação da UI
  void _onRefresh() async {
    await _controller.refresh();
  }

  void _onItemTap(ItemEntity item) {
    // Navegação ou ação específica
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildAppBar(),
      body: CustomListenableBuilder<[State], I[Feature]Failure, [State]>(
        controller: _controller,
        builder: (context, state, isLoading, hasError, error) {
          return _buildContent(state, isLoading, hasError, error);
        },
      ),
      floatingActionButton: _buildFAB(),
    );
  }

  PreferredSizeWidget _buildAppBar() {
    return CustomAppBar(
      title: '[Feature Title]',
      actions: [
        // Actions específicas
      ],
    );
  }

  Widget _buildContent(
    [State] state,
    bool isLoading,
    bool hasError,
    I[Feature]Failure? error,
  ) {
    // Loading inicial
    if (isLoading && state == [initialState]) {
      return const Center(child: CustomLoading());
    }

    // Error inicial
    if (hasError && state == [initialState]) {
      return CognaRequestError(
        error: error!,
        onRetry: _onRefresh,
      );
    }

    // Empty state
    if (state.[isEmpty]) {
      return EmptyStateWidget(
        message: 'Nenhum item encontrado',
        onAction: _onRefresh,
      );
    }

    // Success state
    return _buildSuccessContent(state, isLoading);
  }

  Widget _buildSuccessContent([State] state, bool isLoading) {
    return RefreshIndicator(
      onRefresh: _onRefresh,
      child: ListView.builder(
        controller: _scrollController,
        itemCount: state.items.length + (isLoading ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == state.items.length) {
            return const LoadingIndicator();
          }
          
          return [Item]Tile(
            item: state.items[index],
            onTap: () => _onItemTap(state.items[index]),
          );
        },
      ),
    );
  }

  Widget? _buildFAB() {
    return FloatingActionButton(
      onPressed: () => _onCreateNew(),
      child: const Icon(Icons.add),
    );
  }
}
```

### Convenções de Page

**Nomenclatura:**
- Page: `[Feature]Page` (sempre sufixo Page)
- Args: `[Feature]PageArgs` (parâmetros tipificados)
- State: `_[Feature]PageState` (private state class)

**Estrutura:**
- Args class para parâmetros tipificados
- Controllers injetados via DM.i.get
- CustomListenableBuilder para binding reativo
- Métodos privados para organização de código

**Lifecycle:**
- initState(): Inicialização de dados e UI controllers
- dispose(): Cleanup de todos os controllers
- build(): Composição da UI com tratamento de estados

---

## 📋 Checklist para Pages

### Checklist de Criação ✅

**Estrutura da Page:**
- [ ] Extends StatefulWidget com args tipificados
- [ ] Controllers injetados via DM.i.get no initState
- [ ] CustomListenableBuilder para binding com controllers
- [ ] Localizada em `lib/src/presentation/pages/[feature]/`

**Gerenciamento de Estado:**
- [ ] Usa CustomListenableBuilder consistentemente
- [ ] Trata todos os estados: loading, error, success, empty
- [ ] Error handling com retry appropriado
- [ ] Loading states com shimmer ou indicators

**Navigation e Args:**
- [ ] Args class tipificada para parâmetros
- [ ] Navegação baseada em callbacks ou routes
- [ ] PopScope configurado quando necessário
- [ ] Tratamento de back navigation

**UI Controllers:**
- [ ] ScrollController, TabController, etc. configurados
- [ ] Listeners configurados no initState
- [ ] Dispose de todos os controllers no dispose

**Memory Management:**
- [ ] Dispose de todos os controllers (negócio + UI)
- [ ] Cleanup de listeners e subscriptions
- [ ] Verificação de vazamentos de memória

**User Experience:**
- [ ] Pull-to-refresh implementado onde apropriado
- [ ] Empty states informativos
- [ ] Error states com ações de retry
- [ ] Loading states com feedback visual
- [ ] Navegação fluida entre telas

---

## 🎯 Diretrizes para Pages

### ✅ Boas Práticas

```dart
// ✅ Args tipificados
class UserDetailsPageArgs {
  UserDetailsPageArgs({required this.userId, this.tab});
  
  final String userId;
  final UserTab? tab;
}

// ✅ Controllers organizados
class UserDetailsPage extends StatefulWidget {
  final _userController = DM.i.get<UserDetailsController>();
  final _postsController = DM.i.get<UserPostsController>();
  
  // Setup no initState
  @override
  void initState() {
    super.initState();
    _userController.loadUser(widget.args.userId);
    _postsController.loadPosts(widget.args.userId);
  }
}

// ✅ CustomListenableBuilder consistente
CustomListenableBuilder<UserEntity, IUserFailure, UserEntity>(
  controller: _userController,
  builder: (context, user, isLoading, hasError, error) {
    if (isLoading) return const UserShimmer();
    if (hasError) return UserErrorWidget(error: error!);
    return UserProfileWidget(user: user);
  },
)

// ✅ Tratamento completo de estados
Widget _buildContent(UserState state, bool isLoading, bool hasError, error) {
  // Loading inicial
  if (isLoading && state.users.isEmpty) {
    return const UsersShimmer();
  }
  
  // Error inicial  
  if (hasError && state.users.isEmpty) {
    return ErrorWidget(error: error!, onRetry: _refresh);
  }
  
  // Empty state
  if (state.users.isEmpty) {
    return const EmptyUsersWidget();
  }
  
  // Success com dados
  return UsersList(users: state.users, isLoadingMore: isLoading);
}

// ✅ Dispose completo
@override
void dispose() {
  _userController.dispose();
  _postsController.dispose();
  _scrollController.dispose();
  _tabController.dispose();
  super.dispose();
}
```

### ❌ Evitar

```dart
// ❌ Args não tipificados
class UserDetailsPage extends StatefulWidget {
  const UserDetailsPage({super.key, this.userId}); // String simples
  
  final String? userId;
}

// ❌ Controllers como campos de instância
class _UserDetailsPageState extends State<UserDetailsPage> {
  final userController = UserDetailsController(); // ❌ Instanciação direta
}

// ❌ ValueListenableBuilder ao invés de CustomListenableBuilder
ValueListenableBuilder<UserEntity>(
  valueListenable: controller,
  builder: (context, user, child) {
    // ❌ Precisa verificar estados manualmente
    if (controller.isLoading) return LoadingWidget();
    if (controller.hasError) return ErrorWidget(error: controller.error!);
    return UserWidget(user: user);
  },
)

// ❌ Não tratar todos os estados
Widget build(BuildContext context) {
  return CustomListenableBuilder(
    controller: controller,
    builder: (context, state, isLoading, hasError, error) {
      // ❌ Só trata success, ignora loading/error
      return UsersList(users: state.users);
    },
  );
}

// ❌ Memory leaks
@override
void dispose() {
  // ❌ Esqueceu de fazer dispose dos controllers
  super.dispose();
}

// ❌ Logic de negócio na page
class UserDetailsPage extends StatefulWidget {
  void _calculateUserScore(UserEntity user) {
    // ❌ Lógica de negócio aqui
    final score = user.posts.length * 10 + user.followers.length;
    // ...
  }
}
```

---

## 🚀 Padrões Avançados

### 1. Master-Detail Pattern

```dart
class UsersPage extends StatefulWidget {
  @override
  State<UsersPage> createState() => _UsersPageState();
}

class _UsersPageState extends State<UsersPage> {
  final _usersController = DM.i.get<UsersController>();
  final _selectedUserController = DM.i.get<SelectedUserController>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Row(
        children: [
          // Master: Lista de usuários
          Expanded(
            flex: 1,
            child: CustomListenableBuilder(
              controller: _usersController,
              builder: (context, users, isLoading, hasError, error) {
                return UsersList(
                  users: users.items,
                  selectedUserId: _selectedUserController.state.id,
                  onUserSelected: (user) => _selectedUserController.selectUser(user),
                );
              },
            ),
          ),
          
          // Detail: Detalhes do usuário selecionado
          Expanded(
            flex: 2,
            child: CustomListenableBuilder(
              controller: _selectedUserController,
              builder: (context, user, isLoading, hasError, error) {
                if (user.id.isEmpty) {
                  return const SelectUserPrompt();
                }
                
                return UserDetailsPanel(user: user);
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### 2. Infinite Scroll Pattern

```dart
class InfiniteScrollPage extends StatefulWidget {
  @override
  State<InfiniteScrollPage> createState() => _InfiniteScrollPageState();
}

class _InfiniteScrollPageState extends State<InfiniteScrollPage> {
  final _controller = DM.i.get<InfiniteListController>();
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _controller.loadFirstPage();
    _setupInfiniteScroll();
  }

  void _setupInfiniteScroll() {
    _scrollController.addListener(() {
      if (_scrollController.position.pixels >=
          _scrollController.position.maxScrollExtent - 200) {
        _controller.loadNextPage();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, state, isLoading, hasError, error) {
          return ListView.builder(
            controller: _scrollController,
            itemCount: state.items.length + (state.hasMore ? 1 : 0),
            itemBuilder: (context, index) {
              if (index == state.items.length) {
                if (hasError) {
                  return LoadingErrorWidget(
                    error: error!,
                    onRetry: () => _controller.retryLastPage(),
                  );
                }
                return const LoadingIndicator();
              }
              
              return ItemTile(item: state.items[index]);
            },
          );
        },
      ),
    );
  }
}
```

### 3. Search with Debounce Pattern

```dart
class SearchPage extends StatefulWidget {
  @override
  State<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends State<SearchPage> {
  final _controller = DM.i.get<SearchController>();
  final _searchController = TextEditingController();
  Timer? _debounce;

  @override
  void initState() {
    super.initState();
    _searchController.addListener(_onSearchChanged);
  }

  void _onSearchChanged() {
    if (_debounce?.isActive ?? false) _debounce!.cancel();
    
    _debounce = Timer(const Duration(milliseconds: 500), () {
      final query = _searchController.text.trim();
      if (query.isNotEmpty) {
        _controller.search(query);
      } else {
        _controller.clearSearch();
      }
    });
  }

  @override
  void dispose() {
    _debounce?.cancel();
    _searchController.dispose();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: TextField(
          controller: _searchController,
          decoration: const InputDecoration(
            hintText: 'Pesquisar...',
            border: InputBorder.none,
          ),
        ),
      ),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, state, isLoading, hasError, error) {
          if (_searchController.text.isEmpty) {
            return const SearchPromptWidget();
          }
          
          if (isLoading) {
            return const SearchLoadingWidget();
          }
          
          if (hasError) {
            return SearchErrorWidget(
              error: error!,
              onRetry: () => _controller.search(_searchController.text),
            );
          }
          
          if (state.results.isEmpty) {
            return SearchEmptyWidget(query: _searchController.text);
          }
          
          return SearchResultsList(results: state.results);
        },
      ),
    );
  }
}
```

---

## 🎯 Resumo dos Benefícios

### ✅ **Arquitetura Consistente**
- **Pattern uniformes**: Mesma estrutura para todos os tipos de página
- **CustomListenableBuilder**: Binding reativo padronizado
- **Args tipificados**: Type safety na navegação
- **Error handling**: Tratamento consistente de falhas

### ✅ **Developer Experience**
- **Templates reutilizáveis**: Estrutura padrão para acelerar desenvolvimento
- **Separation of concerns**: Pages apenas compõem, controllers gerenciam estado
- **Memory safety**: Dispose automático previne vazamentos
- **Code organization**: Estrutura clara e previsível

### ✅ **User Experience**
- **Loading states**: Feedback visual imediato
- **Error recovery**: Ações de retry em caso de falha  
- **Empty states**: Orientação clara quando não há dados
- **Responsive interactions**: Pull-to-refresh, infinite scroll

### ✅ **Maintainability**
- **Testable**: Controllers isolados facilitam testes
- **Scalable**: Padrões replicáveis para qualquer feature
- **Flexible**: Suporte a múltiplos controllers por página
- **Clean**: Separação clara entre UI e lógica de negócio

Esta arquitetura de Pages garante **interface consistente**, **experiência fluida** e **código maintível** em aplicações Flutter complexas! 🎯
