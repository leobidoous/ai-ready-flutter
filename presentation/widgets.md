# Presentation Widgets - Clean Architecture

## 📚 Visão Geral

Os **Widgets** na camada de **Presentation** são responsáveis por **componentizar a interface** e **promover reutilização** de código. Seguem princípios rigorosos de **manutenibilidade**, **complexidade controlada** e **composição modular** para manter pages organizadas e escaláveis.

### 🎯 Princípios Fundamentais dos Widgets

**O QUE os Widgets FAZEM:**
- ✅ **Componentizam Interface**: Quebram complexity em partes menores
- ✅ **Promovem Reutilização**: Widgets compartilhados entre múltiplas pages
- ✅ **Controlam Complexidade**: Mantêm pages com máximo 300 linhas
- ✅ **Organizam Código**: Part/part of para componentes específicos
- ✅ **Encapsulam Responsabilidade**: Cada widget tem propósito único e claro

**O QUE os Widgets NÃO FAZEM:**
- ❌ **Não contêm lógica de negócio**: Apenas apresentação e UI
- ❌ **Não fazem chamadas diretas**: Recebem dados via parâmetros
- ❌ **Não gerenciam estado global**: Apenas estado local da UI
- ❌ **Não fazem navegação direta**: Usam callbacks para coordenação
- ❌ **Não duplicam funcionalidade**: Evitam código repetitivo

### 🏗️ Estratégias de Componentização

```
lib/src/presentation/
├── widgets/                         # Widgets reutilizáveis globais
│   ├── common/                     # Componentes básicos
│   │   ├── custom_button.dart      # Botões padronizados
│   │   ├── custom_input.dart       # Inputs personalizados
│   │   └── custom_card.dart        # Cards reutilizáveis
│   ├── forms/                      # Componentes de formulário
│   │   ├── address_form.dart       # Formulário de endereço
│   │   └── payment_form.dart       # Formulário de pagamento
│   └── lists/                      # Componentes de listagem
│       ├── user_list_item.dart     # Item de usuário
│       └── product_list_item.dart  # Item de produto
└── pages/
    └── feature/
        ├── feature_page.dart       # Page principal (< 300 linhas)
        └── widgets/                # Widgets específicos da page
            ├── feature_header.dart # Parte específica (part of)
            └── feature_content.dart # Parte específica (part of)
```

---

## 🎯 Regras de Componentização

### 📏 **1. Limite de Linhas em Pages**

**Regra Obrigatória:** Pages devem ter **máximo 300 linhas** (ideal 200-300)

```dart
// ✅ Page bem organizada (250 linhas)
class UserProfilePage extends StatefulWidget {
  const UserProfilePage({super.key, required this.args});

  final UserProfilePageArgs args;

  @override
  State<UserProfilePage> createState() => _UserProfilePageState();
}

class _UserProfilePageState extends State<UserProfilePage> {
  final _controller = DM.i.get<UserProfileController>();

  @override
  void initState() {
    super.initState();
    _loadUserData();
  }

  void _loadUserData() {
    _controller.loadUser(widget.args.userId);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildAppBar(),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, user, isLoading, hasError, error) {
          if (isLoading) return const _UserProfileShimmer();
          if (hasError) return _UserProfileError(error: error!, onRetry: _loadUserData);
          return _UserProfileContent(user: user);
        },
      ),
    );
  }

  PreferredSizeWidget _buildAppBar() {
    return CustomAppBar(
      title: 'Perfil do Usuário',
      actions: [
        IconButton(
          onPressed: () => _editUser(),
          icon: const Icon(Icons.edit),
        ),
      ],
    );
  }

  void _editUser() {
    Nav.to.pushNamed(
      UserRoutes.edit,
      arguments: UserEditPageArgs(userId: widget.args.userId),
    );
  }
}
```

### 🔄 **2. Widgets Reutilizáveis Globais**

**Quando usar:** Componentes utilizados em **múltiplas pages**

```dart
// lib/src/presentation/widgets/common/custom_button.dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

/// Botão personalizado reutilizável em toda aplicação
/// 
/// Padroniza apparência e comportamento dos botões
/// seguindo design system da aplicação.
class CustomButton extends StatelessWidget {
  const CustomButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.type = CustomButtonType.primary,
    this.size = CustomButtonSize.medium,
    this.isLoading = false,
    this.isEnabled = true,
    this.icon,
  });

  final String label;
  final VoidCallback? onPressed;
  final CustomButtonType type;
  final CustomButtonSize size;
  final bool isLoading;
  final bool isEnabled;
  final IconData? icon;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: size.height,
      child: ElevatedButton(
        onPressed: isEnabled && !isLoading ? onPressed : null,
        style: _buildButtonStyle(context),
        child: isLoading 
            ? _buildLoadingIndicator()
            : _buildButtonContent(),
      ),
    );
  }

  ButtonStyle _buildButtonStyle(BuildContext context) {
    return ElevatedButton.styleFrom(
      backgroundColor: type.getBackgroundColor(context),
      foregroundColor: type.getForegroundColor(context),
      padding: EdgeInsets.symmetric(
        horizontal: size.horizontalPadding,
        vertical: size.verticalPadding,
      ),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(size.borderRadius),
      ),
    );
  }

  Widget _buildLoadingIndicator() {
    return SizedBox(
      height: 16,
      width: 16,
      child: CircularProgressIndicator(
        strokeWidth: 2,
        valueColor: AlwaysStoppedAnimation(type.getForegroundColor(context)),
      ),
    );
  }

  Widget _buildButtonContent() {
    if (icon != null) {
      return Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, size: size.iconSize),
          SizedBox(width: Spacing.xs.value),
          Text(label, style: size.textStyle),
        ],
      );
    }
    
    return Text(label, style: size.textStyle);
  }
}

enum CustomButtonType {
  primary,
  secondary,
  outline,
  danger;

  Color getBackgroundColor(BuildContext context) {
    switch (this) {
      case CustomButtonType.primary:
        return context.colorScheme.primary;
      case CustomButtonType.secondary:
        return context.colorScheme.secondary;
      case CustomButtonType.outline:
        return Colors.transparent;
      case CustomButtonType.danger:
        return context.colorScheme.error;
    }
  }

  Color getForegroundColor(BuildContext context) {
    switch (this) {
      case CustomButtonType.primary:
      case CustomButtonType.secondary:
      case CustomButtonType.danger:
        return context.colorScheme.onPrimary;
      case CustomButtonType.outline:
        return context.colorScheme.primary;
    }
  }
}

enum CustomButtonSize {
  small,
  medium,
  large;

  double get height {
    switch (this) {
      case CustomButtonSize.small: return 32;
      case CustomButtonSize.medium: return 40;
      case CustomButtonSize.large: return 48;
    }
  }

  double get horizontalPadding {
    switch (this) {
      case CustomButtonSize.small: return 12;
      case CustomButtonSize.medium: return 16;
      case CustomButtonSize.large: return 20;
    }
  }

  double get verticalPadding {
    switch (this) {
      case CustomButtonSize.small: return 6;
      case CustomButtonSize.medium: return 8;
      case CustomButtonSize.large: return 12;
    }
  }

  double get borderRadius {
    switch (this) {
      case CustomButtonSize.small: return 6;
      case CustomButtonSize.medium: return 8;
      case CustomButtonSize.large: return 10;
    }
  }

  double get iconSize {
    switch (this) {
      case CustomButtonSize.small: return 14;
      case CustomButtonSize.medium: return 16;
      case CustomButtonSize.large: return 18;
    }
  }

  TextStyle get textStyle {
    switch (this) {
      case CustomButtonSize.small: 
        return const TextStyle(fontSize: 12, fontWeight: FontWeight.w500);
      case CustomButtonSize.medium: 
        return const TextStyle(fontSize: 14, fontWeight: FontWeight.w500);
      case CustomButtonSize.large: 
        return const TextStyle(fontSize: 16, fontWeight: FontWeight.w600);
    }
  }
}
```

### 📦 **3. Part/Part of para Componentes Específicos**

**Quando usar:** Page ficando grande, mas widget é **específico daquela page**

```dart
// lib/src/presentation/pages/enrollment_details/enrollment_details_page.dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

import '../../controllers/enrollment_details_controller.dart';

part 'widgets/enrollment_candidate_card.dart';
part 'widgets/enrollment_offer_card.dart';
part 'widgets/enrollment_expiration_timer.dart';
part 'widgets/enrollment_steps.dart';

/// Page de detalhes da matrícula
/// 
/// Mostra informações completas sobre a matrícula,
/// organizando em componentes específicos via part/part of.
class EnrollmentDetailsPage extends StatefulWidget {
  const EnrollmentDetailsPage({super.key, required this.args});

  final EnrollmentDetailsPageArgs args;

  @override
  State<EnrollmentDetailsPage> createState() => _EnrollmentDetailsPageState();
}

class _EnrollmentDetailsPageState extends State<EnrollmentDetailsPage> {
  final _controller = DM.i.get<EnrollmentDetailsController>();

  @override
  void initState() {
    super.initState();
    _controller.getEnrollmentDetails(idOrigin: widget.args.idOrigin);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Detalhes da Matrícula'),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, enrollment, isLoading, hasError, error) {
          if (isLoading) {
            return const Center(child: CustomLoading());
          }
          
          if (hasError) {
            return CognaRequestError(
              error: error!,
              onRetry: () => _controller.getEnrollmentDetails(
                idOrigin: widget.args.idOrigin,
              ),
            );
          }

          return CustomScrollContent(
            padding: EdgeInsets.all(Spacing.md.value),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                _EnrollmentCandidateCard(enrollment: enrollment),
                Spacing.md.vertical,
                _EnrollmentOfferCard(enrollment: enrollment),
                Spacing.md.vertical,
                _EnrollmentExpirationTimer(enrollment: enrollment),
                Spacing.md.vertical,
                _EnrollmentSteps(enrollment: enrollment),
              ],
            ),
          );
        },
      ),
    );
  }
}

class EnrollmentDetailsPageArgs {
  EnrollmentDetailsPageArgs({required this.idOrigin});
  
  final String idOrigin;
}
```

```dart
// lib/src/presentation/pages/enrollment_details/widgets/enrollment_candidate_card.dart
part of '../enrollment_details_page.dart';

/// Card que exibe informações do candidato
/// 
/// Componente específico da page de detalhes da matrícula,
/// organizado via part of para reduzir complexity da page principal.
class _EnrollmentCandidateCard extends StatelessWidget {
  const _EnrollmentCandidateCard({required this.enrollment});

  final EnrollmentEntity enrollment;

  @override
  Widget build(BuildContext context) {
    return CustomCard(
      child: Padding(
        padding: EdgeInsets.all(Spacing.md.value),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(
                  Icons.person_outline,
                  color: context.colorScheme.primary,
                  size: 20,
                ),
                SizedBox(width: Spacing.xs.value),
                Text(
                  'Dados do Candidato',
                  style: context.textTheme.titleMedium?.copyWith(
                    fontWeight: FontWeight.w600,
                  ),
                ),
              ],
            ),
            Spacing.sm.vertical,
            _buildCandidateInfo('Nome', enrollment.candidate.name),
            _buildCandidateInfo('CPF', enrollment.candidate.cpf.masked),
            _buildCandidateInfo('Email', enrollment.candidate.email),
            _buildCandidateInfo('Telefone', enrollment.candidate.phone.masked),
            if (enrollment.candidate.birthDate != null)
              _buildCandidateInfo(
                'Data de Nascimento',
                enrollment.candidate.birthDate!.format('dd/MM/yyyy'),
              ),
          ],
        ),
      ),
    );
  }

  Widget _buildCandidateInfo(String label, String value) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: Spacing.xs.value),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 120,
            child: Text(
              '$label:',
              style: TextStyle(
                fontWeight: FontWeight.w500,
                color: Colors.grey[600],
              ),
            ),
          ),
          Expanded(
            child: Text(
              value.isNotEmpty ? value : 'Não informado',
              style: TextStyle(
                fontWeight: FontWeight.w400,
                color: value.isNotEmpty ? null : Colors.grey[400],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

### 🔒 **4. Widgets Privados para Page**

**Quando usar:** Componentes pequenos **específicos de uma page**

```dart
// Dentro da page (não em arquivo separado)
class UserListPage extends StatefulWidget {
  const UserListPage({super.key});

  @override
  State<UserListPage> createState() => _UserListPageState();
}

class _UserListPageState extends State<UserListPage> {
  final _controller = DM.i.get<UserListController>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Usuários'),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, users, isLoading, hasError, error) {
          if (isLoading) return const _UserListShimmer();
          if (hasError) return _UserListError(error: error!, onRetry: _controller.refresh);
          if (users.isEmpty) return const _UserListEmpty();
          
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) => _UserListItem(
              user: users[index],
              onTap: () => _openUserDetails(users[index]),
            ),
          );
        },
      ),
    );
  }

  void _openUserDetails(UserEntity user) {
    Nav.to.pushNamed(
      UserRoutes.details,
      arguments: UserDetailsPageArgs(userId: user.id),
    );
  }
}

/// Widget privado para shimmer da lista de usuários
/// 
/// Específico da UserListPage, usado apenas aqui.
class _UserListShimmer extends StatelessWidget {
  const _UserListShimmer();

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 10,
      itemBuilder: (context, index) => Padding(
        padding: EdgeInsets.symmetric(
          horizontal: Spacing.md.value,
          vertical: Spacing.sm.value,
        ),
        child: CustomShimmer(
          child: Card(
            child: Container(
              height: 80,
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(8),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

/// Widget privado para item da lista de usuários
/// 
/// Específico da UserListPage, usado apenas aqui.
class _UserListItem extends StatelessWidget {
  const _UserListItem({
    required this.user,
    required this.onTap,
  });

  final UserEntity user;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(
        horizontal: Spacing.md.value,
        vertical: Spacing.sm.value,
      ),
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: user.avatarUrl.isNotEmpty 
              ? NetworkImage(user.avatarUrl)
              : null,
          child: user.avatarUrl.isEmpty 
              ? Text(user.initials)
              : null,
        ),
        title: Text(
          user.name,
          style: context.textTheme.titleMedium,
        ),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(user.email),
            SizedBox(height: Spacing.xs.value),
            _UserStatusChip(status: user.status),
          ],
        ),
        trailing: const Icon(Icons.chevron_right),
        onTap: onTap,
      ),
    );
  }
}

/// Widget privado para chip de status do usuário
/// 
/// Específico da UserListPage, usado apenas aqui.
class _UserStatusChip extends StatelessWidget {
  const _UserStatusChip({required this.status});

  final UserStatus status;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.symmetric(
        horizontal: Spacing.xs.value,
        vertical: 2,
      ),
      decoration: BoxDecoration(
        color: status.color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(
          color: status.color.withOpacity(0.3),
          width: 1,
        ),
      ),
      child: Text(
        status.label,
        style: TextStyle(
          fontSize: 12,
          fontWeight: FontWeight.w500,
          color: status.color,
        ),
      ),
    );
  }
}

/// Widget privado para estado vazio da lista
/// 
/// Específico da UserListPage, usado apenas aqui.
class _UserListEmpty extends StatelessWidget {
  const _UserListEmpty();

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            Icons.people_outline,
            size: 64,
            color: Colors.grey[400],
          ),
          SizedBox(height: Spacing.md.value),
          Text(
            'Nenhum usuário encontrado',
            style: context.textTheme.titleMedium?.copyWith(
              color: Colors.grey[600],
            ),
          ),
          SizedBox(height: Spacing.sm.value),
          Text(
            'Adicione usuários para começar',
            style: context.textTheme.bodyMedium?.copyWith(
              color: Colors.grey[500],
            ),
          ),
        ],
      ),
    );
  }
}

/// Widget privado para erro da lista
/// 
/// Específico da UserListPage, usado apenas aqui.
class _UserListError extends StatelessWidget {
  const _UserListError({
    required this.error,
    required this.onRetry,
  });

  final IUserFailure error;
  final VoidCallback onRetry;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: EdgeInsets.all(Spacing.md.value),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.error_outline,
              size: 64,
              color: context.colorScheme.error,
            ),
            SizedBox(height: Spacing.md.value),
            Text(
              'Erro ao carregar usuários',
              style: context.textTheme.titleMedium,
              textAlign: TextAlign.center,
            ),
            SizedBox(height: Spacing.sm.value),
            Text(
              error.message,
              style: context.textTheme.bodyMedium?.copyWith(
                color: Colors.grey[600],
              ),
              textAlign: TextAlign.center,
            ),
            SizedBox(height: Spacing.md.value),
            CustomButton(
              label: 'Tentar Novamente',
              onPressed: onRetry,
              icon: Icons.refresh,
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🎨 Padrões de Implementação

### 1. Widget Reutilizável com Configuração

```dart
// lib/src/presentation/widgets/forms/address_form_widget.dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

/// Widget de formulário de endereço reutilizável
/// 
/// Pode ser usado em diferentes contexts (cadastro, edição, checkout)
/// com configuração flexível de campos obrigatórios.
class AddressFormWidget extends StatefulWidget {
  const AddressFormWidget({
    super.key,
    required this.onAddressChanged,
    this.initialAddress,
    this.requiredFields = const {},
    this.enabledFields = const {},
    this.autoFocus = false,
  });

  final Function(AddressEntity) onAddressChanged;
  final AddressEntity? initialAddress;
  final Set<AddressField> requiredFields;
  final Set<AddressField> enabledFields;
  final bool autoFocus;

  @override
  State<AddressFormWidget> createState() => _AddressFormWidgetState();
}

class _AddressFormWidgetState extends State<AddressFormWidget> {
  final _formKey = GlobalKey<FormState>();
  final _zipCodeController = TextEditingController();
  final _streetController = TextEditingController();
  final _numberController = TextEditingController();
  final _complementController = TextEditingController();
  final _neighborhoodController = TextEditingController();
  final _cityController = TextEditingController();
  final _stateController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _initializeControllers();
    _setupListeners();
  }

  void _initializeControllers() {
    if (widget.initialAddress != null) {
      final address = widget.initialAddress!;
      _zipCodeController.text = address.zipCode;
      _streetController.text = address.street;
      _numberController.text = address.number;
      _complementController.text = address.complement;
      _neighborhoodController.text = address.neighborhood;
      _cityController.text = address.city;
      _stateController.text = address.state;
    }
  }

  void _setupListeners() {
    _zipCodeController.addListener(_onAddressChanged);
    _streetController.addListener(_onAddressChanged);
    _numberController.addListener(_onAddressChanged);
    _complementController.addListener(_onAddressChanged);
    _neighborhoodController.addListener(_onAddressChanged);
    _cityController.addListener(_onAddressChanged);
    _stateController.addListener(_onAddressChanged);
  }

  void _onAddressChanged() {
    final address = AddressEntity(
      zipCode: _zipCodeController.text,
      street: _streetController.text,
      number: _numberController.text,
      complement: _complementController.text,
      neighborhood: _neighborhoodController.text,
      city: _cityController.text,
      state: _stateController.text,
    );
    
    widget.onAddressChanged(address);
  }

  @override
  void dispose() {
    _zipCodeController.dispose();
    _streetController.dispose();
    _numberController.dispose();
    _complementController.dispose();
    _neighborhoodController.dispose();
    _cityController.dispose();
    _stateController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          if (_isFieldEnabled(AddressField.zipCode))
            CustomTextFormField(
              controller: _zipCodeController,
              labelText: 'CEP',
              hintText: '00000-000',
              keyboardType: TextInputType.number,
              autofocus: widget.autoFocus,
              validator: _isFieldRequired(AddressField.zipCode)
                  ? (value) => _validateZipCode(value)
                  : null,
              onChanged: (_) => _onZipCodeChanged(),
            ),
          
          SizedBox(height: Spacing.md.value),
          
          if (_isFieldEnabled(AddressField.street))
            CustomTextFormField(
              controller: _streetController,
              labelText: 'Rua',
              hintText: 'Nome da rua',
              textCapitalization: TextCapitalization.words,
              validator: _isFieldRequired(AddressField.street)
                  ? (value) => _validateRequired(value, 'Rua é obrigatória')
                  : null,
            ),
          
          SizedBox(height: Spacing.md.value),
          
          Row(
            children: [
              if (_isFieldEnabled(AddressField.number))
                Expanded(
                  flex: 2,
                  child: CustomTextFormField(
                    controller: _numberController,
                    labelText: 'Número',
                    hintText: '123',
                    keyboardType: TextInputType.number,
                    validator: _isFieldRequired(AddressField.number)
                        ? (value) => _validateRequired(value, 'Número é obrigatório')
                        : null,
                  ),
                ),
              
              if (_isFieldEnabled(AddressField.number) && 
                  _isFieldEnabled(AddressField.complement))
                SizedBox(width: Spacing.md.value),
              
              if (_isFieldEnabled(AddressField.complement))
                Expanded(
                  flex: 3,
                  child: CustomTextFormField(
                    controller: _complementController,
                    labelText: 'Complemento',
                    hintText: 'Apto, bloco, etc.',
                    textCapitalization: TextCapitalization.words,
                  ),
                ),
            ],
          ),
          
          // Outros campos seguem o mesmo padrão...
        ],
      ),
    );
  }

  bool _isFieldEnabled(AddressField field) {
    return widget.enabledFields.isEmpty || widget.enabledFields.contains(field);
  }

  bool _isFieldRequired(AddressField field) {
    return widget.requiredFields.contains(field);
  }

  String? _validateRequired(String? value, String message) {
    if (value?.trim().isEmpty ?? true) {
      return message;
    }
    return null;
  }

  String? _validateZipCode(String? value) {
    if (_isFieldRequired(AddressField.zipCode)) {
      if (value?.trim().isEmpty ?? true) {
        return 'CEP é obrigatório';
      }
      if (!RegExp(r'^\d{5}-?\d{3}$').hasMatch(value!)) {
        return 'CEP inválido';
      }
    }
    return null;
  }

  void _onZipCodeChanged() {
    // Lógica para buscar endereço por CEP
    // Poderia chamar um controller ou callback
  }
}

enum AddressField {
  zipCode,
  street,
  number,
  complement,
  neighborhood,
  city,
  state,
}
```

### 2. Widget Part/Part of com Estados Complexos

```dart
// lib/src/presentation/pages/checkout/checkout_page.dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

part 'widgets/checkout_header.dart';
part 'widgets/checkout_items_summary.dart';
part 'widgets/checkout_payment_section.dart';
part 'widgets/checkout_address_section.dart';
part 'widgets/checkout_total_section.dart';

class CheckoutPage extends StatefulWidget {
  const CheckoutPage({super.key, required this.args});

  final CheckoutPageArgs args;

  @override
  State<CheckoutPage> createState() => _CheckoutPageState();
}

class _CheckoutPageState extends State<CheckoutPage> {
  final _controller = DM.i.get<CheckoutController>();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Finalizar Pedido'),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, checkout, isLoading, hasError, error) {
          return CustomScrollView(
            slivers: [
              SliverToBoxAdapter(child: _CheckoutHeader(checkout: checkout)),
              SliverToBoxAdapter(child: _CheckoutItemsSummary(checkout: checkout)),
              SliverToBoxAdapter(child: _CheckoutAddressSection(checkout: checkout)),
              SliverToBoxAdapter(child: _CheckoutPaymentSection(checkout: checkout)),
              SliverToBoxAdapter(child: _CheckoutTotalSection(checkout: checkout)),
            ],
          );
        },
      ),
    );
  }
}
```

```dart
// lib/src/presentation/pages/checkout/widgets/checkout_payment_section.dart
part of '../checkout_page.dart';

/// Seção de pagamento do checkout
/// 
/// Componente específico do checkout que gerencia
/// seleção de métodos de pagamento e dados do cartão.
class _CheckoutPaymentSection extends StatefulWidget {
  const _CheckoutPaymentSection({required this.checkout});

  final CheckoutEntity checkout;

  @override
  State<_CheckoutPaymentSection> createState() => _CheckoutPaymentSectionState();
}

class _CheckoutPaymentSectionState extends State<_CheckoutPaymentSection> {
  PaymentMethod? _selectedPaymentMethod;
  
  @override
  void initState() {
    super.initState();
    _selectedPaymentMethod = widget.checkout.paymentMethod;
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(Spacing.md.value),
      child: Padding(
        padding: EdgeInsets.all(Spacing.md.value),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _buildSectionHeader(),
            Spacing.md.vertical,
            _buildPaymentMethods(),
            if (_selectedPaymentMethod == PaymentMethod.creditCard) ...[
              Spacing.md.vertical,
              _buildCreditCardForm(),
            ],
          ],
        ),
      ),
    );
  }

  Widget _buildSectionHeader() {
    return Row(
      children: [
        Icon(
          Icons.payment_outlined,
          color: context.colorScheme.primary,
        ),
        SizedBox(width: Spacing.xs.value),
        Text(
          'Forma de Pagamento',
          style: context.textTheme.titleMedium?.copyWith(
            fontWeight: FontWeight.w600,
          ),
        ),
      ],
    );
  }

  Widget _buildPaymentMethods() {
    return Column(
      children: PaymentMethod.values.map((method) {
        return _PaymentMethodTile(
          method: method,
          isSelected: _selectedPaymentMethod == method,
          onSelected: (selected) {
            setState(() {
              _selectedPaymentMethod = selected ? method : null;
            });
            _updateCheckoutPayment();
          },
        );
      }).toList(),
    );
  }

  Widget _buildCreditCardForm() {
    return _CreditCardForm(
      initialData: widget.checkout.creditCardData,
      onChanged: (cardData) {
        // Atualizar dados do cartão
        _updateCheckoutPayment();
      },
    );
  }

  void _updateCheckoutPayment() {
    // Atualizar controller com nova forma de pagamento
    final controller = DM.i.get<CheckoutController>();
    controller.updatePaymentMethod(_selectedPaymentMethod);
  }
}

/// Tile para seleção de método de pagamento
class _PaymentMethodTile extends StatelessWidget {
  const _PaymentMethodTile({
    required this.method,
    required this.isSelected,
    required this.onSelected,
  });

  final PaymentMethod method;
  final bool isSelected;
  final Function(bool) onSelected;

  @override
  Widget build(BuildContext context) {
    return CheckboxListTile(
      value: isSelected,
      onChanged: onSelected,
      title: Row(
        children: [
          Icon(method.icon, size: 20),
          SizedBox(width: Spacing.xs.value),
          Text(method.label),
        ],
      ),
      subtitle: method.hasDescription 
          ? Text(method.description)
          : null,
      contentPadding: EdgeInsets.zero,
    );
  }
}

/// Formulário para dados do cartão de crédito
class _CreditCardForm extends StatefulWidget {
  const _CreditCardForm({
    required this.onChanged,
    this.initialData,
  });

  final Function(CreditCardData) onChanged;
  final CreditCardData? initialData;

  @override
  State<_CreditCardForm> createState() => _CreditCardFormState();
}

class _CreditCardFormState extends State<_CreditCardForm> {
  final _numberController = TextEditingController();
  final _holderController = TextEditingController();
  final _expiryController = TextEditingController();
  final _cvvController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _initializeControllers();
    _setupListeners();
  }

  void _initializeControllers() {
    if (widget.initialData != null) {
      final data = widget.initialData!;
      _numberController.text = data.maskedNumber;
      _holderController.text = data.holderName;
      _expiryController.text = data.expiryDate;
      _cvvController.text = data.cvv;
    }
  }

  void _setupListeners() {
    _numberController.addListener(_onFormChanged);
    _holderController.addListener(_onFormChanged);
    _expiryController.addListener(_onFormChanged);
    _cvvController.addListener(_onFormChanged);
  }

  void _onFormChanged() {
    final cardData = CreditCardData(
      number: _numberController.text,
      holderName: _holderController.text,
      expiryDate: _expiryController.text,
      cvv: _cvvController.text,
    );
    
    widget.onChanged(cardData);
  }

  @override
  void dispose() {
    _numberController.dispose();
    _holderController.dispose();
    _expiryController.dispose();
    _cvvController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        CustomTextFormField(
          controller: _numberController,
          labelText: 'Número do Cartão',
          hintText: '0000 0000 0000 0000',
          keyboardType: TextInputType.number,
          validator: _validateCardNumber,
        ),
        SizedBox(height: Spacing.md.value),
        CustomTextFormField(
          controller: _holderController,
          labelText: 'Nome no Cartão',
          hintText: 'Nome completo',
          textCapitalization: TextCapitalization.words,
          validator: _validateHolderName,
        ),
        SizedBox(height: Spacing.md.value),
        Row(
          children: [
            Expanded(
              child: CustomTextFormField(
                controller: _expiryController,
                labelText: 'Validade',
                hintText: 'MM/AA',
                keyboardType: TextInputType.number,
                validator: _validateExpiryDate,
              ),
            ),
            SizedBox(width: Spacing.md.value),
            Expanded(
              child: CustomTextFormField(
                controller: _cvvController,
                labelText: 'CVV',
                hintText: '123',
                keyboardType: TextInputType.number,
                obscureText: true,
                validator: _validateCVV,
              ),
            ),
          ],
        ),
      ],
    );
  }

  String? _validateCardNumber(String? value) {
    if (value?.isEmpty ?? true) {
      return 'Número do cartão é obrigatório';
    }
    if (value!.replaceAll(' ', '').length < 16) {
      return 'Número do cartão inválido';
    }
    return null;
  }

  String? _validateHolderName(String? value) {
    if (value?.trim().isEmpty ?? true) {
      return 'Nome no cartão é obrigatório';
    }
    return null;
  }

  String? _validateExpiryDate(String? value) {
    if (value?.isEmpty ?? true) {
      return 'Validade é obrigatória';
    }
    if (!RegExp(r'^\d{2}/\d{2}$').hasMatch(value!)) {
      return 'Formato inválido (MM/AA)';
    }
    return null;
  }

  String? _validateCVV(String? value) {
    if (value?.isEmpty ?? true) {
      return 'CVV é obrigatório';
    }
    if (value!.length < 3) {
      return 'CVV deve ter 3 dígitos';
    }
    return null;
  }
}
```

---

## 📋 Templates para Widgets

### Template - Widget Reutilizável Global

```dart
// lib/src/presentation/widgets/[category]/[widget_name].dart
import 'package:base_core/base_core.dart';
import 'package:flutter/material.dart';

/// [Descrição do widget e sua funcionalidade]
/// 
/// [Explicação de quando usar e configurações disponíveis]
/// [Exemplo de uso básico]
class [WidgetName] extends StatelessWidget {
  const [WidgetName]({
    super.key,
    required this.requiredParam,
    this.optionalParam,
    this.configuration = const [WidgetName]Configuration(),
  });

  final RequiredType requiredParam;
  final OptionalType? optionalParam;
  final [WidgetName]Configuration configuration;

  @override
  Widget build(BuildContext context) {
    return Container(
      // Implementação do widget
      child: _buildContent(context),
    );
  }

  Widget _buildContent(BuildContext context) {
    // Lógica de construção do widget
    return Column(
      children: [
        // Elementos do widget
      ],
    );
  }
}

/// Configuração para customização do widget
class [WidgetName]Configuration {
  const [WidgetName]Configuration({
    this.backgroundColor,
    this.textStyle,
    this.padding,
    this.borderRadius = 8.0,
  });

  final Color? backgroundColor;
  final TextStyle? textStyle;
  final EdgeInsets? padding;
  final double borderRadius;
}
```

### Template - Widget Part/Part of

```dart
// Na page principal:
part 'widgets/[widget_name].dart';

// No arquivo do widget:
// lib/src/presentation/pages/[page]/widgets/[widget_name].dart
part of '../[page_name]_page.dart';

/// [Descrição do componente específico]
/// 
/// Componente específico da [PageName]Page,
/// organizado via part of para reduzir complexity da page principal.
class _[WidgetName] extends StatelessWidget {
  const _[WidgetName]({
    required this.data,
    this.onAction,
  });

  final DataType data;
  final VoidCallback? onAction;

  @override
  Widget build(BuildContext context) {
    return Container(
      // Implementação específica
      child: _buildSpecificContent(),
    );
  }

  Widget _buildSpecificContent() {
    // Lógica específica do componente
    return Column(
      children: [
        // Elementos específicos
      ],
    );
  }
}
```

### Convenções de Widgets

**Nomenclatura:**
- Widgets globais: `CustomButton`, `AddressFormWidget`
- Widgets part/part of: `_ComponentName` (underscore + PascalCase)
- Widgets privados: `_WidgetName` (underscore + descriptive name)

**Estrutura:**
- Required params primeiro, optional depois
- Configuration objects para customização
- Callbacks para ações e eventos
- Build methods privados para organização

**Organização:**
- Widgets globais: `/widgets/[category]/`
- Part widgets: `/pages/[page]/widgets/`
- Documentação clara do propósito
- Validações quando necessário

---

## 📋 Checklist para Widgets

### Checklist de Criação ✅

**Estrutura do Widget:**
- [ ] Escolha do tipo correto: Global, Part/Part of, ou Privado
- [ ] Nomenclatura seguindo padrão estabelecido
- [ ] Localização apropriada na estrutura de pastas
- [ ] Documentação clara da responsabilidade

**Widget Reutilizável Global:**
- [ ] Implementado em `/widgets/[category]/`
- [ ] Usado em múltiplas pages ou contextos
- [ ] Configuration object para customização
- [ ] Parâmetros obrigatórios vs opcionais bem definidos
- [ ] Callbacks para ações externas

**Widget Part/Part of:**
- [ ] Part declaration correto na page principal
- [ ] Widget específico para uma page grande (>250 linhas)
- [ ] Nome com underscore (privado)
- [ ] Implementação em `/pages/[page]/widgets/`

**Widget Privado:**
- [ ] Dentro da própria page (não arquivo separado)
- [ ] Específico para uma única page
- [ ] Nome com underscore (privado)
- [ ] Componente pequeno e simples

**Complexity Management:**
- [ ] Page principal com máximo 300 linhas
- [ ] Widgets quebram complexity em partes menores
- [ ] Cada widget tem responsabilidade única
- [ ] Não duplica código entre widgets

**Performance e UX:**
- [ ] Widget otimizado (const constructors quando possível)
- [ ] Rebuild mínimo necessário
- [ ] Estados de loading/error tratados
- [ ] Acessibilidade considerada

---

## 🎯 Diretrizes para Widgets

### ✅ Boas Práticas

```dart
// ✅ Widget reutilizável bem estruturado
class CustomInputField extends StatelessWidget {
  const CustomInputField({
    super.key,
    required this.controller,
    required this.labelText,
    this.hintText,
    this.validator,
    this.keyboardType = TextInputType.text,
    this.obscureText = false,
    this.enabled = true,
  });

  final TextEditingController controller;
  final String labelText;
  final String? hintText;
  final String? Function(String?)? validator;
  final TextInputType keyboardType;
  final bool obscureText;
  final bool enabled;
}

// ✅ Widget part/part of bem organizado
part of '../user_profile_page.dart';

class _UserInfoCard extends StatelessWidget {
  const _UserInfoCard({required this.user});

  final UserEntity user;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(Spacing.md.value),
        child: _buildUserInfo(),
      ),
    );
  }
}

// ✅ Widget privado específico
class _UserAvatarWidget extends StatelessWidget {
  const _UserAvatarWidget({required this.user});

  final UserEntity user;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      radius: 40,
      backgroundImage: user.avatarUrl.isNotEmpty 
          ? NetworkImage(user.avatarUrl)
          : null,
      child: user.avatarUrl.isEmpty 
          ? Text(user.initials)
          : null,
    );
  }
}

// ✅ Configuration object para customização
class ButtonConfiguration {
  const ButtonConfiguration({
    this.style = ButtonStyle.primary,
    this.size = ButtonSize.medium,
    this.borderRadius = 8.0,
  });

  final ButtonStyle style;
  final ButtonSize size;
  final double borderRadius;
}

// ✅ Callbacks para ações
class UserListItem extends StatelessWidget {
  const UserListItem({
    super.key,
    required this.user,
    required this.onTap,
    this.onEdit,
    this.onDelete,
  });

  final UserEntity user;
  final VoidCallback onTap;
  final VoidCallback? onEdit;
  final VoidCallback? onDelete;
}
```

### ❌ Evitar

```dart
// ❌ Widget muito específico em pasta global
// lib/src/presentation/widgets/common/user_profile_avatar.dart
class UserProfileAvatar extends StatelessWidget {
  // ❌ Específico demais para ser global
}

// ❌ Page muito grande sem componentização
class UserPage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // ❌ 400+ linhas de widgets inline
          Container(/* muitos widgets... */),
          // ❌ Código repetitivo não componentizado
        ],
      ),
    );
  }
}

// ❌ Widget part/part of para algo simples
part of '../simple_page.dart';

class _TinyWidget extends StatelessWidget {
  // ❌ Muito simples para ser part of
  @override
  Widget build(BuildContext context) {
    return Text('Hello');
  }
}

// ❌ Widget sem configuração/flexibilidade
class FixedButton extends StatelessWidget {
  // ❌ Hardcoded demais, não reutilizável
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blue, // ❌ Hardcoded
        padding: const EdgeInsets.all(16), // ❌ Não configurável
      ),
      onPressed: () {
        // ❌ Ação hardcoded, não flexível
        Navigator.pop(context);
      },
      child: const Text('Voltar'), // ❌ Texto fixo
    );
  }
}

// ❌ Widget com muitas responsabilidades
class ComplexWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ❌ Faz networking
        FutureBuilder<User>(
          future: ApiService.getUser(), // ❌ Acesso direto a API
          builder: (context, snapshot) {
            // ❌ Business logic no widget
            if (snapshot.hasData) {
              final user = snapshot.data!;
              // ❌ Validações de negócio aqui
              if (user.isActive && user.hasPermission) {
                // ❌ Muito código, deveria ser componentizado
                return Container(/* 100+ linhas */);
              }
            }
            return Container();
          },
        ),
        // ❌ Mais responsabilidades misturadas...
      ],
    );
  }
}
```

---

## 🚀 Uso dos Widgets

### 1. Composição em Pages

```dart
class ProductDetailsPage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(title: 'Detalhes do Produto'),
      body: CustomListenableBuilder(
        controller: _controller,
        builder: (context, product, isLoading, hasError, error) {
          if (isLoading) return const ProductDetailsShimmer();
          if (hasError) return ProductErrorWidget(error: error!);
          
          return CustomScrollContent(
            child: Column(
              children: [
                // Widget reutilizável global
                ProductImageCarousel(
                  images: product.images,
                  onImageTap: _openImageGallery,
                ),
                
                // Widget part/part of
                _ProductInfoSection(product: product),
                
                // Widget reutilizável global
                PriceDisplayWidget(
                  price: product.price,
                  discount: product.discount,
                  configuration: PriceDisplayConfiguration(
                    showDiscount: true,
                    style: PriceStyle.large,
                  ),
                ),
                
                // Widget part/part of
                _ProductActionsSection(
                  product: product,
                  onAddToCart: _addToCart,
                  onBuyNow: _buyNow,
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

### 2. Configuração de Widgets Globais

```dart
// Uso básico
CustomButton(
  label: 'Salvar',
  onPressed: _save,
)

// Uso com configuração
CustomButton(
  label: 'Cancelar',
  onPressed: _cancel,
  type: CustomButtonType.outline,
  size: CustomButtonSize.small,
)

// Uso avançado
CustomButton(
  label: 'Processar Pagamento',
  onPressed: _processPayment,
  type: CustomButtonType.primary,
  size: CustomButtonSize.large,
  isLoading: _isProcessing,
  icon: Icons.payment,
)
```

### 3. Part/Part of para Pages Grandes

```dart
// checkout_page.dart (230 linhas)
part 'widgets/checkout_header.dart';
part 'widgets/checkout_summary.dart';
part 'widgets/checkout_payment.dart';
part 'widgets/checkout_shipping.dart';

class CheckoutPage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          _CheckoutHeader(order: _order),
          _CheckoutSummary(items: _order.items),
          _CheckoutPayment(onPaymentChanged: _updatePayment),
          _CheckoutShipping(onShippingChanged: _updateShipping),
        ],
      ),
    );
  }
}
```

---

## 🎯 Padrões Avançados

### 1. Widget Factory Pattern

```dart
class WidgetFactory {
  static Widget createButton({
    required String label,
    required VoidCallback onPressed,
    ButtonType type = ButtonType.primary,
  }) {
    switch (type) {
      case ButtonType.primary:
        return CustomButton(
          label: label,
          onPressed: onPressed,
          type: CustomButtonType.primary,
        );
      case ButtonType.secondary:
        return OutlinedButton(
          onPressed: onPressed,
          child: Text(label),
        );
      case ButtonType.text:
        return TextButton(
          onPressed: onPressed,
          child: Text(label),
        );
    }
  }

  static Widget createListItem({
    required String title,
    String? subtitle,
    Widget? leading,
    VoidCallback? onTap,
  }) {
    return ListTile(
      title: Text(title),
      subtitle: subtitle != null ? Text(subtitle) : null,
      leading: leading,
      onTap: onTap,
      trailing: onTap != null ? const Icon(Icons.chevron_right) : null,
    );
  }
}
```

### 2. Builder Pattern para Widgets Complexos

```dart
class FormBuilder {
  final List<Widget> _fields = [];
  EdgeInsets _padding = EdgeInsets.all(16);
  CrossAxisAlignment _crossAxisAlignment = CrossAxisAlignment.stretch;

  FormBuilder addField(Widget field) {
    _fields.add(field);
    return this;
  }

  FormBuilder addTextField({
    required TextEditingController controller,
    required String label,
    String? hint,
    TextInputType? keyboardType,
    String? Function(String?)? validator,
  }) {
    _fields.add(
      CustomTextFormField(
        controller: controller,
        labelText: label,
        hintText: hint,
        keyboardType: keyboardType ?? TextInputType.text,
        validator: validator,
      ),
    );
    return this;
  }

  FormBuilder addSpacing([double? height]) {
    _fields.add(SizedBox(height: height ?? 16));
    return this;
  }

  FormBuilder setPadding(EdgeInsets padding) {
    _padding = padding;
    return this;
  }

  FormBuilder setCrossAxisAlignment(CrossAxisAlignment alignment) {
    _crossAxisAlignment = alignment;
    return this;
  }

  Widget build() {
    return Padding(
      padding: _padding,
      child: Column(
        crossAxisAlignment: _crossAxisAlignment,
        children: _fields,
      ),
    );
  }
}

// Uso:
FormBuilder()
  .addTextField(
    controller: _nameController,
    label: 'Nome',
    validator: (value) => value?.isEmpty == true ? 'Campo obrigatório' : null,
  )
  .addSpacing()
  .addTextField(
    controller: _emailController,
    label: 'Email',
    keyboardType: TextInputType.emailAddress,
  )
  .addSpacing(24)
  .addField(
    CustomButton(
      label: 'Salvar',
      onPressed: _save,
    ),
  )
  .build();
```

### 3. Conditional Widget Rendering

```dart
class ConditionalWidget extends StatelessWidget {
  const ConditionalWidget({
    super.key,
    required this.condition,
    required this.child,
    this.fallback,
  });

  final bool condition;
  final Widget child;
  final Widget? fallback;

  @override
  Widget build(BuildContext context) {
    if (condition) {
      return child;
    } else if (fallback != null) {
      return fallback!;
    } else {
      return const SizedBox.shrink();
    }
  }
}

// Uso:
ConditionalWidget(
  condition: user.isVip,
  child: VipBadgeWidget(),
  fallback: RegularUserWidget(),
)
```

---

## 🎯 Resumo dos Benefícios

### ✅ **Manutenibilidade**
- **Pages organizadas**: Máximo 300 linhas mantém código legível
- **Componentização clara**: Cada widget tem responsabilidade única
- **Part/Part of**: Organização sem duplicação para widgets específicos
- **Reutilização**: Widgets globais eliminam duplicação de código

### ✅ **Scalabilidade**
- **Widgets modulares**: Fácil adição de novas funcionalidades
- **Configuration objects**: Customização sem quebrar API
- **Composition over inheritance**: Flexibilidade na construção de UIs
- **Factory patterns**: Criação consistente de widgets complexos

### ✅ **Developer Experience**
- **Templates claros**: Estrutura padrão acelera desenvolvimento
- **Naming conventions**: Fácil identificação do tipo de widget
- **Documentation**: Propósito e uso bem documentados
- **Type safety**: Parâmetros tipificados previnem erros

### ✅ **Performance**
- **Const constructors**: Otimização automática do Flutter
- **Rebuild otimizado**: Widgets específicos minimizam rebuilds
- **Memory efficiency**: Reutilização reduz instanciação
- **Load balancing**: Complexity distribuída em componentes menores

Esta arquitetura de Widgets garante **código organizado**, **reutilização eficiente** e **manutenibilidade** em aplicações Flutter complexas! 🎯
