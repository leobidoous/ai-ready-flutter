# Presentation Controllers - Clean Architecture

## 📚 Visão Geral

Os **Controllers** na camada de **Presentation** são responsáveis por **gerenciar estado da UI** e **coordenar operações de negócio**. Baseados no `ValueNotifier` do Flutter, eles integram o pattern `Either` para oferecer gerenciamento de estado robusto e type-safe.

### 🎯 Princípios Fundamentais dos Controllers

**O QUE os Controllers FAZEM:**
- ✅ **Gerenciam Estado da UI**: State management reativo com `ValueNotifier`
- ✅ **Coordenam UseCases**: Bridge entre UI e regras de negócio
- ✅ **Gerenciam Loading/Error**: Estados auxiliares automáticos
- ✅ **Integram Either Pattern**: Type-safe error handling
- ✅ **Sincronizam Serviços**: Coordenação de múltiplos serviços externos

**O QUE os Controllers NÃO FAZEM:**
- ❌ **Não contêm regras de negócio**: Apenas coordenam UseCases
- ❌ **Não fazem comunicação direta**: Usam UseCases para acessar dados
- ❌ **Não implementam protocolos**: Delegam para camadas apropriadas
- ❌ **Não contêm lógica de transformação**: Recebem dados prontos
- ❌ **Não gerenciam persistência**: Usam repositories via UseCases

### 🏗️ Localização e Estrutura

```
lib/src/presentation/controllers/
├── app_controller.dart          # Estado global da aplicação
├── session_controller.dart      # Estado do usuário logado
├── login_controller.dart        # Fluxo de autenticação
└── feature_controller.dart      # Controllers específicos por feature
```

---

## 🔍 Anatomia do CustomController

### Estrutura Base

```dart
import 'package:base_core/base_core.dart';

abstract class CustomController<E, S> extends ValueNotifier<S> {
  CustomController(super.value);

  // Estados auxiliares
  bool _wasDisposed = false;
  final ValueNotifier<E?> _error = ValueNotifier(null);
  final ValueNotifier<bool> _loading = ValueNotifier(false);

  // Getters de estado
  bool get wasDisposed => _wasDisposed;
  E? get error => _error.value;
  bool get hasError => _error.value != null;
  bool get isLoading => _loading.value;
  S get state => value;

  // Gerenciamento de erro
  void setError(E value) { /* ... */ }
  void clearError({bool update = false}) { /* ... */ }

  // Gerenciamento de loading
  void setLoading(bool value, {bool update = true}) { /* ... */ }

  // Atualização de estado
  void update(S state, {force = false}) { /* ... */ }

  // Execução com Either pattern
  Future<Either<E, S>> execute(
    Future<Either<E, S>> Function() function, {
    force = false,
  }) async { /* ... */ }

  @override
  void dispose() { /* cleanup automático */ }
}
```

### 🔑 Elementos Essenciais

1. **Generics `<E, S>`**: E = Error/Failure, S = Success/State
2. **ValueNotifier Integration**: Reatividade nativa do Flutter
3. **Estados Auxiliares**: loading, error, hasError automáticos
4. **Execute Method**: Wrapper para Either pattern automatizado
5. **Memory Safety**: Dispose automático e verificação wasDisposed

---

## 📚 Exemplos Práticos Reais

### 1. AppController - Estado Global

```dart
class AppController extends CustomController<IAppFailure, Unit> {
  AppController({
    required this.sentryService,
    required this.locationService,
    required this.localUserUsecase,
    required this.permissionService,
    required this.preferencesController,
    required this.firebaseAnalyticsService,
    required this.datadogCrashlyticsService,
    required this.firebaseNotificationsService,
  }) : super(unit);

  late SessionEntity session;
  final ILocationService locationService;
  final ILocalUserUsecase localUserUsecase;
  final IPermissionService permissionService;
  final ISentryCrashlyticsService sentryService;
  final IFirebaseAnalyticsService firebaseAnalyticsService;
  final IDatadogCrashlyticsService datadogCrashlyticsService;
  final UserLocalPreferencesController preferencesController;
  final IFirebaseNotificationsService firebaseNotificationsService;

  Future<Either<IAppFailure, SessionEntity>> getSession() async {
    final response = await localUserUsecase.getLocalSession();
    return response.fold(
      (l) => Left(AppUnknownError(message: '$l')),
      (session) async {
        this.session = session;
        if (!localUserUsecase.sessionIsValid(session: session)) {
          return Left(AppUnknownError(
            message: 'Sua sessão está inválida. Realize o login novamente.',
          ));
        }
        return Right(session);
      },
    );
  }

  Future<Either<IAppFailure, SessionEntity>> onLogin(
    SessionEntity session,
  ) async {
    return setSession(session: session).then((onValue) {
      return onValue.fold(Left, (session) async {
        this.session = session;
        final user = UserModel.fromEntity(session.user);

        // Coordena múltiplos serviços
        _identifySentryUser(user);
        _identifyDatadogUser(user, session);
        _identifyFirebaseUser(user, session);
        
        await preferencesController.getUserPreferences(session.user.id);
        return Right(session);
      });
    });
  }

  @override
  void dispose() {
    preferencesController.dispose();
    super.dispose();
  }
}
```

### 2. SessionController - Estado do Usuário

```dart
class SessionController extends CustomController<IUserFailure, SessionEntity> {
  SessionController(
    super.value, {
    required this.localUserUsecase,
    required this.userUsecase,
  });

  final ILocalUserUsecase localUserUsecase;
  final IUserUsecase userUsecase;

  TokenEntity get token => state.token;
  UserEntity get user => state.user;

  Future<Either<IUserFailure, SessionEntity>> updateSession({
    TokenEntity? token,
    UserEntity? user,
  }) async {
    setLoading(true);
    final session = state.copyWith(token: token, user: user);
    
    if (localUserUsecase.sessionIsValid(session: session)) {
      final response = await localUserUsecase.setSession(session: session);
      return response.fold(
        (l) {
          setError(UserUnknownError(message: l.toString()));
          return Left(error!);
        },
        (r) {
          update(session);
          return Right(state);
        },
      );
    } else {
      setError(UserUnknownError(message: 'Sessão inválida!'));
      return Left(error!);
    }
  }

  Future<void> getUser() async {
    await execute(
      () => userUsecase.getLoggedUser().then(
        (value) => value.fold(
          (l) => Left(l),
          (r) async {
            await updateSession(user: r);
            return Right(state.copyWith(user: r));
          },
        ),
      ),
    );
  }
}
```

### 3. LoginController - Fluxo de Autenticação

```dart
class LoginController extends CustomController<ILoginFailure, SessionEntity> {
  LoginController({
    required this.onLogin,
    required this.loginUsecase,
  }) : super(SessionModel.fromMap({}));

  final ILoginUsecase loginUsecase;
  final Future<Either<ICustomFailure, SessionEntity>> Function(SessionEntity) onLogin;

  Future<void> login({
    required String login,
    required String password,
  }) async {
    await execute(() async {
      final data = LoginRequestDataEntity(
        login: login,
        password: password,
        deviceLocation: PositionModel.fromMap({}),
      );
      
      return loginUsecase.login(loginData: data).then(
        (value) => value.fold(
          (l) => Left(l),
          (r) => onLogin(r).then(
            (onValue) => onValue.fold(
              (l) => Left(LoginUnknownError(message: l.message)),
              Right,
            ),
          ),
        ),
      );
    });
  }
}
```

### 4. MainController - Estado de UI/Navegação

```dart
class MainController extends CustomController<Exception, BottomNavBarType> {
  MainController() : super(BottomNavBarType.home);

  Size _bottomNavBarSize = Size.zero;
  Size get bottomNavBarSize => _bottomNavBarSize;
  set bottomNavBarSize(Size? size) {
    if (size == null) return;
    _bottomNavBarSize = size;
    update(state, force: true);
  }

  final _scaffoldKey = GlobalKey<ScaffoldState>();
  GlobalKey<ScaffoldState> get scaffoldKey => _scaffoldKey;

  void onChangeTypeByPath(String path) {
    if (path.contains(HomeRoutes.root.completePath)) {
      update(BottomNavBarType.home);
    } else if (path.contains(FunnelRoutes.root.completePath)) {
      update(BottomNavBarType.funnel);
    }
  }

  bool canShowClipboardBSheet = true;

  Future<String?> getClipboardData() async {
    canShowClipboardBSheet = false;
    return Clipboard.getData(Clipboard.kTextPlain).then((value) async {
      return value?.text;
    });
  }

  bool _balancesIsVisible = false;
  bool get balanceIsVisible => _balancesIsVisible;

  void onChangedBalancesVisibility(bool value) {
    _balancesIsVisible = value;
    update(state, force: true);
  }
}
```

---

## 🎨 Padrões de Implementação

### 1. Controller com Estado Simples

```dart
class UserProfileController extends CustomController<IUserFailure, UserEntity> {
  UserProfileController({
    required this.userUsecase,
  }) : super(UserEntity.empty());

  final IUserUsecase userUsecase;

  Future<void> loadProfile() async {
    await execute(() => userUsecase.getLoggedUser());
  }

  Future<void> updateProfile(UserEntity updatedUser) async {
    await execute(() => userUsecase.updateUser(user: updatedUser));
  }
}
```

### 2. Controller com Estado Complexo

```dart
class OrderListController extends CustomController<IOrderFailure, OrderListState> {
  OrderListController({
    required this.orderUsecase,
  }) : super(OrderListState.initial());

  final IOrderUsecase orderUsecase;

  Future<void> loadOrders({bool isRefresh = false}) async {
    if (isRefresh) {
      update(state.copyWith(page: 1, orders: []));
    }
    
    await execute(() async {
      final result = await orderUsecase.getOrders(
        page: state.page,
        limit: state.limit,
      );
      
      return result.fold(
        Left,
        (orders) {
          final allOrders = isRefresh ? orders : [...state.orders, ...orders];
          final newState = state.copyWith(
            orders: allOrders,
            page: state.page + 1,
            hasMore: orders.length == state.limit,
          );
          return Right(newState);
        },
      );
    });
  }
}
```

### 3. Controller com Callback Injection

```dart
class CheckoutController extends CustomController<ICheckoutFailure, CheckoutState> {
  CheckoutController({
    required this.checkoutUsecase,
    required this.onSuccess,
    required this.onError,
  }) : super(CheckoutState.initial());

  final ICheckoutUsecase checkoutUsecase;
  final VoidCallback onSuccess;
  final Function(ICheckoutFailure) onError;

  Future<void> processCheckout(CheckoutDataEntity data) async {
    await execute(() async {
      final result = await checkoutUsecase.processOrder(data: data);
      
      return result.fold(
        (failure) {
          onError(failure);
          return Left(failure);
        },
        (order) {
          onSuccess();
          return Right(state.copyWith(completedOrder: order));
        },
      );
    });
  }
}
```

### 4. Controller com Coordenação de Serviços

```dart
class NotificationController extends CustomController<INotificationFailure, NotificationState> {
  NotificationController({
    required this.notificationUsecase,
    required this.analyticsService,
    required this.localNotificationService,
  }) : super(NotificationState.initial());

  final INotificationUsecase notificationUsecase;
  final IAnalyticsService analyticsService;
  final ILocalNotificationService localNotificationService;

  Future<void> markAsRead(String notificationId) async {
    await execute(() async {
      // Track analytics
      analyticsService.track('notification_read', {'id': notificationId});
      
      // Update local notification
      await localNotificationService.markAsRead(notificationId);
      
      // Update remote
      final result = await notificationUsecase.markAsRead(id: notificationId);
      
      return result.fold(
        Left,
        (notification) {
          final updatedNotifications = state.notifications.map((n) {
            return n.id == notificationId ? notification : n;
          }).toList();
          
          return Right(state.copyWith(notifications: updatedNotifications));
        },
      );
    });
  }
}
```

---

## 📋 Template para Controllers

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart';

/// Controller que gerencia [descrição do estado/feature]
/// 
/// Responsável por [lista de responsabilidades]
/// e coordena [lista de operações].
class [Nome]Controller extends CustomController<I[Nome]Failure, [Estado]> {
  [Nome]Controller({
    required this.[usecase],
    // outros dependencies
  }) : super([estadoInicial]);

  final I[Nome]Usecase [usecase];
  // outros dependencies injetados

  // Getters convenientes do estado
  [TipoPropriedade] get [propriedade] => state.[propriedade];

  /// [Descrição da operação principal]
  /// 
  /// [parâmetro] descrição do parâmetro
  /// 
  /// Executa [descrição do que faz] e atualiza o estado
  Future<void> [operacaoPrincipal]([parâmetros]) async {
    await execute(() async {
      // Lógica de coordenação
      final result = await [usecase].[metodo]([parâmetros]);
      
      return result.fold(
        Left, // Propagação automática de erro
        (data) {
          // Transformação/atualização do estado
          final newState = state.copyWith([propriedades]);
          return Right(newState);
        },
      );
    });
  }

  /// [Operação secundária se necessário]
  Future<void> [operacaoSecundaria]([parâmetros]) async {
    // Implementação
  }

  @override
  void dispose() {
    // Cleanup de dependencies se necessário
    super.dispose();
  }
}
```

### Convenções de Controller

**Nomenclatura:**
- Controller: `[Feature]Controller` (sempre sufixo Controller)
- Métodos: camelCase descritivos da ação
- Estados: `[Feature]State` ou Entity diretamente

**Estrutura:**
- Dependencies injetados via construtor
- Estado inicial definido no super()
- Getters convenientes para propriedades do estado
- Métodos públicos para operações da UI

**Either Integration:**
- Sempre usar `execute()` para operações assíncronas
- Failures propagados automaticamente
- Success states atualizados via `Right(newState)`

---

## 📋 Checklist para Controllers

### Checklist de Criação ✅

**Estrutura do Controller:**
- [ ] Estende `CustomController<FailureType, StateType>`
- [ ] Dependencies injetados via construtor
- [ ] Estado inicial apropriado no super()
- [ ] Localizado em `lib/src/presentation/controllers/`

**Gerenciamento de Estado:**
- [ ] Usa `execute()` para operações com Either
- [ ] Estados atualizados via `update()` ou `Right()`
- [ ] Getters convenientes para propriedades do estado
- [ ] Loading/Error gerenciados automaticamente

**Integration com UseCases:**
- [ ] Dependency injection de IUseCases apenas
- [ ] Não contém regras de negócio
- [ ] Delega toda lógica para UseCases
- [ ] Trata apenas coordenação e estado da UI

**Integration com Flutter UI:**
- [ ] Usa `CustomListenableBuilder` para binding com UI
- [ ] Generics corretos `<Entity, Failure, State>` definidos
- [ ] Trata todos os estados: loading, error, success
- [ ] Error handling com retry appropriado
- [ ] Empty states implementados quando necessário

**Memory Management:**
- [ ] Override de `dispose()` se necessário
- [ ] Cleanup de dependencies manuais
- [ ] Verificação `wasDisposed` em operações críticas
- [ ] Não cria memory leaks

**Documentação:**
- [ ] Descrição clara da responsabilidade
- [ ] Documentação dos métodos principais
- [ ] Exemplos de uso quando complexo
- [ ] Comentários em lógica não óbvia

---

## 🎯 Diretrizes para Controllers

### ✅ Boas Práticas com CustomListenableBuilder

```dart
// ✅ Type safety completo
CustomListenableBuilder<UserEntity, IUserFailure, UserEntity>(
  controller: userController,
  builder: (context, state, isLoading, hasError, error) {
    // Todos os tipos são inferidos corretamente
  },
)

// ✅ Tratamento de estados bem definido
builder: (context, state, isLoading, hasError, error) {
  // 1. Loading primeiro (se dados ainda não carregaram)
  if (isLoading && state == initialState) {
    return const LoadingWidget();
  }
  
  // 2. Error com possibilidade de retry
  if (hasError) {
    return ErrorWidget(
      error: error!,
      onRetry: () => controller.retry(),
    );
  }
  
  // 3. Success state com dados válidos
  return SuccessWidget(data: state);
}

// ✅ Combinação de loading + dados (pagination, refresh)
builder: (context, state, isLoading, hasError, error) {
  return Column(
    children: [
      // Dados existentes sempre visíveis
      Expanded(child: DataList(items: state.items)),
      
      // Loading indicator para operações adicionais
      if (isLoading)
        const Padding(
          padding: EdgeInsets.all(16),
          child: CircularProgressIndicator(),
        ),
    ],
  );
}

// ✅ Error handling granular
builder: (context, state, isLoading, hasError, error) {
  if (hasError) {
    // Diferentes tratamentos por tipo de erro
    switch (error.runtimeType) {
      case NetworkError:
        return NetworkErrorWidget(onRetry: controller.retry);
      case ValidationError:
        return ValidationErrorWidget(error: error as ValidationError);
      default:
        return GenericErrorWidget(error: error!);
    }
  }
  
  return SuccessWidget(data: state);
}
```

### ❌ Evitar com CustomListenableBuilder

```dart
// ❌ Não verificar tipos corretamente
CustomListenableBuilder(
  controller: controller, // ❌ Sem generics
  builder: (context, state, isLoading, hasError, error) {
    // Types não são inferidos
  },
)

// ❌ Logic complexa no builder
builder: (context, state, isLoading, hasError, error) {
  // ❌ Muito processamento no builder
  final processedData = heavyProcessing(state);
  final filteredData = complexFiltering(processedData);
  
  return Widget(data: filteredData);
}

// ❌ Não tratar todos os estados
builder: (context, state, isLoading, hasError, error) {
  // ❌ Só trata success, ignora loading/error
  return DataWidget(data: state);
}

// ❌ Rebuild desnecessário
builder: (context, state, isLoading, hasError, error) {
  return Column(
    children: [
      // ❌ Cria widgets desnecessariamente a cada rebuild
      ExpensiveWidget(),
      DataWidget(data: state),
    ],
  );
}
```

### 🎯 Padrões Recomendados

#### 1. **Empty State Pattern**

```dart
CustomListenableBuilder<UserListState, IUserFailure, UserListState>(
  controller: controller,
  builder: (context, state, isLoading, hasError, error) {
    // Loading inicial
    if (isLoading && state.users.isEmpty) {
      return const LoadingWidget();
    }
    
    // Error com lista vazia
    if (hasError && state.users.isEmpty) {
      return ErrorWidget(error: error!, onRetry: controller.loadUsers);
    }
    
    // Empty state
    if (!isLoading && !hasError && state.users.isEmpty) {
      return const EmptyStateWidget(
        message: 'Nenhum usuário encontrado',
        action: 'Adicionar Usuário',
        onAction: controller.addUser,
      );
    }
    
    // Success com dados
    return UserListWidget(
      users: state.users,
      isLoadingMore: isLoading,
      onLoadMore: controller.loadMoreUsers,
    );
  },
)
```

#### 2. **Optimistic Updates Pattern**

```dart
CustomListenableBuilder<TodoListState, ITodoFailure, TodoListState>(
  controller: controller,
  builder: (context, state, isLoading, hasError, error) {
    return ListView.builder(
      itemCount: state.todos.length,
      itemBuilder: (context, index) {
        final todo = state.todos[index];
        
        return TodoTile(
          todo: todo,
          isUpdating: isLoading && state.updatingId == todo.id,
          onToggle: () => controller.toggleTodo(todo.id),
          onDelete: () => controller.deleteTodo(todo.id),
        );
      },
    );
  },
)
```

#### 3. **Multi-state Operations Pattern**

```dart
CustomListenableBuilder<CheckoutState, ICheckoutFailure, CheckoutState>(
  controller: controller,
  builder: (context, state, isLoading, hasError, error) {
    return Column(
      children: [
        // Form sempre visível
        CheckoutForm(
          data: state.formData,
          onUpdate: controller.updateForm,
          enabled: !isLoading,
        ),
        
        // Progress indicator durante checkout
        if (isLoading)
          CheckoutProgressWidget(step: state.currentStep),
        
        // Error message
        if (hasError)
          ErrorBanner(
            error: error!,
            onDismiss: controller.clearError,
          ),
        
        // Submit button
        CheckoutButton(
          enabled: !isLoading && state.isFormValid,
          onPressed: controller.processCheckout,
        ),
      ],
    );
  },
)
```

### ❌ Evitar

```dart
// ❌ Regras de negócio no controller
class UserController extends CustomController<IUserFailure, UserEntity> {
  Future<void> updateUser(UserEntity user) async {
    // ❌ Validação de negócio aqui
    if (user.age < 18) {
      setError(UserValidationError('Menor de idade'));
      return;
    }
    // ...
  }
}

// ❌ Comunicação direta com DataSources
class UserController extends CustomController<IUserFailure, UserEntity> {
  UserController({required this.userDatasource}); // ❌ DataSource direto
  
  final IUserDatasource userDatasource;
}

// ❌ Não usar execute() para Either operations
Future<void> getUser() async {
  setLoading(true);
  final result = await userUsecase.getUser();
  result.fold(
    (l) => setError(l),
    (r) => update(r),
  );
  setLoading(false); // ❌ Manual demais
}

// ❌ Estado mutável diretamente
class BadController extends CustomController<Exception, UserEntity> {
  UserEntity _user = UserEntity.empty();
  
  void updateUserName(String name) {
    _user.name = name; // ❌ Mutação direta
    notifyListeners(); // ❌ Manual
  }
}

// ❌ Memory leaks
@override
void dispose() {
  // ❌ Não limpar resources
  // someSubscription.cancel(); // esqueceu
  super.dispose();
}
```

---

## 🚀 Uso na UI com Flutter

### 1. CustomListenableBuilder (Recomendado) ⭐

```dart
import 'package:base_core/base_core.dart';

class UserProfilePage extends StatelessWidget {
  const UserProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    final controller = context.read<UserProfileController>();
    
    return Scaffold(
      body: CustomListenableBuilder<UserEntity, IUserFailure, UserEntity>(
        controller: controller,
        builder: (context, state, isLoading, hasError, error) {
          // Loading state
          if (isLoading) {
            return const Center(child: CircularProgressIndicator());
          }
          
          // Error state
          if (hasError) {
            return ErrorWidget(
              error: error!,
              onRetry: () => controller.loadProfile(),
            );
          }
          
          // Success state
          return UserProfileWidget(user: state);
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => controller.loadProfile(),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

**Vantagens do CustomListenableBuilder:**
- ✅ **All-in-one**: state, loading, error em um único builder
- ✅ **Type Safety**: Generics garantem tipos corretos
- ✅ **Menos Boilerplate**: Não precisa verificar estados manualmente
- ✅ **Optimized**: Rebuilds apenas quando necessário
- ✅ **Consistent**: Padrão uniforme em toda aplicação

### 2. ValueListenableBuilder (Alternativa)

```dart
class UserProfilePage extends StatelessWidget {
  const UserProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    final controller = context.read<UserProfileController>();
    
    return Scaffold(
      body: ValueListenableBuilder<UserEntity>(
        valueListenable: controller,
        builder: (context, user, child) {
          return Column(
            children: [
              // Loading state
              if (controller.isLoading)
                const CircularProgressIndicator(),
              
              // Error state
              if (controller.hasError)
                ErrorWidget(error: controller.error!),
              
              // Success state
              if (!controller.isLoading && !controller.hasError)
                UserProfileWidget(user: user),
            ],
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => controller.loadProfile(),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
}
```

}
```

### 3. AnimatedBuilder para Multiple States

```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.read<UserProfileController>();
    
    return Scaffold(
      body: AnimatedBuilder(
        animation: Listenable.merge([controller, controller._error, controller._loading]),
        builder: (context, child) {
          // Acesso a todos os estados: loading, error, state
          return _buildContent(controller);
        },
      ),
    );
  }
}
```

### 4. Consumer Pattern (com Provider)

```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<UserProfileController>(
      builder: (context, controller, child) {
        return Scaffold(
          body: controller.isLoading
              ? const LoadingWidget()
              : controller.hasError
                  ? ErrorWidget(error: controller.error!)
                  : UserProfileWidget(user: controller.state),
        );
      },
    );
  }
}
```

---

## 🎨 CustomListenableBuilder - Implementação

### Estrutura do Widget Auxiliar

```dart
import 'package:flutter/material.dart';
import '../../domain/interfaces/custom_controller.dart';

class CustomListenableBuilder<T, E, S> extends StatelessWidget {
  const CustomListenableBuilder({
    super.key,
    required this.controller,
    required this.builder,
  });

  final CustomController<E, S> controller;
  final Widget Function(
    BuildContext context,
    S state,
    bool isLoading,
    bool hasError,
    E? error,
  ) builder;

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder(
      valueListenable: controller,
      builder: (context, state, child) {
        return builder(
          context,
          state,
          controller.isLoading,
          controller.hasError,
          controller.error,
        );
      },
    );
  }
}
```

### 🔑 Características do CustomListenableBuilder

1. **Generics Completos**: `<T, E, S>` para type safety total
2. **Builder Unificado**: Todos os estados em um único callback
3. **ValueNotifier Base**: Usa ValueNotifier internamente para performance
4. **Zero Boilerplate**: Não precisa verificar estados manualmente
5. **Reactividade Otimizada**: Rebuilds automáticos e eficientes

### 🎯 Exemplos Avançados com CustomListenableBuilder

#### Lista com Paginação

```dart
class OrderListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.read<OrderListController>();
    
    return Scaffold(
      body: CustomListenableBuilder<OrderListState, IOrderFailure, OrderListState>(
        controller: controller,
        builder: (context, state, isLoading, hasError, error) {
          if (hasError && state.orders.isEmpty) {
            return ErrorWidget(
              error: error!,
              onRetry: () => controller.loadOrders(isRefresh: true),
            );
          }
          
          return RefreshIndicator(
            onRefresh: () => controller.loadOrders(isRefresh: true),
            child: ListView.builder(
              itemCount: state.orders.length + (isLoading ? 1 : 0),
              itemBuilder: (context, index) {
                if (index == state.orders.length) {
                  return const LoadingIndicator(); // Loading pagination
                }
                
                return OrderTile(
                  order: state.orders[index],
                  onTap: () => _openOrderDetails(state.orders[index]),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

#### Form com Validação

```dart
class UserFormPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.read<UserFormController>();
    
    return Scaffold(
      body: CustomListenableBuilder<UserFormState, IUserFailure, UserFormState>(
        controller: controller,
        builder: (context, state, isLoading, hasError, error) {
          return Form(
            child: Column(
              children: [
                // Form fields
                TextFormField(
                  initialValue: state.name,
                  onChanged: controller.updateName,
                  decoration: InputDecoration(
                    labelText: 'Nome',
                    errorText: state.nameError,
                  ),
                ),
                
                TextFormField(
                  initialValue: state.email,
                  onChanged: controller.updateEmail,
                  decoration: InputDecoration(
                    labelText: 'Email',
                    errorText: state.emailError,
                  ),
                ),
                
                // Error message
                if (hasError)
                  ErrorMessage(error: error!),
                
                // Submit button
                ElevatedButton(
                  onPressed: isLoading || !state.isValid 
                      ? null 
                      : controller.submitForm,
                  child: isLoading 
                      ? const CircularProgressIndicator()
                      : const Text('Salvar'),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.read<UserProfileController>();
    
    return Scaffold(
      body: AnimatedBuilder(
        animation: Listenable.merge([controller, controller._error, controller._loading]),
        builder: (context, child) {
          // Acesso a todos os estados: loading, error, state
          return _buildContent(controller);
        },
      ),
    );
  }
}
```

### 3. Consumer Pattern (com Provider)

```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<UserProfileController>(
      builder: (context, controller, child) {
        return Scaffold(
          body: controller.isLoading
              ? const LoadingWidget()
              : controller.hasError
                  ? ErrorWidget(error: controller.error!)
                  : UserProfileWidget(user: controller.state),
        );
      },
    );
  }
}
```

---

## 🔄 Integração com Clean Architecture

### Fluxo Controller → UseCase → Repository

```dart
// 1. UI chama Controller
onPressed: () => userController.updateProfile(newUser);

// 2. Controller coordena UseCase
class UserController extends CustomController<IUserFailure, UserEntity> {
  Future<void> updateProfile(UserEntity user) async {
    await execute(() => userUsecase.updateUser(user: user));
    //                     ↓
    //              3. UseCase aplica regras de negócio
  }
}

// 3. UseCase implementa regras
class UserUsecase extends IUserUsecase {
  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity user}) async {
    // Validações de negócio
    final validation = _validateUser(user);
    if (validation.isLeft()) return validation;
    
    // Delega para Repository
    return repository.updateUser(user: user);
    //           ↓
    //    4. Repository coordena dados
  }
}
```

### Dependency Injection Setup

```dart
// GetIt registration
void setupControllers() {
  // Controllers
  getIt.registerFactory<UserProfileController>(
    () => UserProfileController(
      userUsecase: getIt<IUserUsecase>(),
    ),
  );
  
  getIt.registerLazySingleton<AppController>(
    () => AppController(
      sentryService: getIt<ISentryCrashlyticsService>(),
      locationService: getIt<ILocationService>(),
      localUserUsecase: getIt<ILocalUserUsecase>(),
      // ... outros services
    ),
  );
}

// Provider setup
MultiProvider(
  providers: [
    ChangeNotifierProvider<AppController>(
      create: (_) => getIt<AppController>(),
    ),
    ChangeNotifierProvider<UserProfileController>(
      create: (_) => getIt<UserProfileController>(),
    ),
  ],
  child: MyApp(),
)
```

---

## 🎯 Resumo dos Benefícios

### ✅ **Reatividade Nativa**
- **ValueNotifier integration**: Reatividade otimizada do Flutter
- **Automatic rebuilds**: UI atualiza automaticamente com mudanças de estado
- **Performance**: Apenas widgets que dependem do estado são reconstruídos

### ✅ **Type Safety com Either**
- **Compile-time safety**: Erros capturados em tempo de compilação
- **Explicit error handling**: Tratamento obrigatório de falhas
- **Consistent patterns**: Mesmo padrão em toda aplicação

### ✅ **Developer Experience**
- **Auto loading/error**: Estados auxiliares gerenciados automaticamente
- **Execute wrapper**: Simplifica operações Either
- **Memory safety**: Dispose automático e verificações de segurança
- **CustomListenableBuilder**: Widget unificado para todos os estados da UI

### ✅ **UI Integration**
- **Zero boilerplate**: CustomListenableBuilder elimina verificações manuais
- **Type safety**: Generics garantem tipos corretos em tempo de compilação
- **Consistent patterns**: Mesmo padrão de UI em toda aplicação
- **Optimized rebuilds**: Apenas widgets necessários são reconstruídos

### ✅ **Clean Architecture Integration**
- **Separation of concerns**: Controllers apenas coordenam, não implementam
- **Dependency inversion**: Dependem de abstrações (IUseCases)
- **Testability**: Fácil mock de dependencies

### ✅ **Production Ready**
- **Memory leak prevention**: Dispose automático de resources
- **Service coordination**: Integração com analytics, crashlytics, etc.
- **Callback injection**: Coordenação flexível entre features

Esta arquitetura de Controllers com **CustomListenableBuilder** garante **estado consistente**, **performance otimizada**, **zero boilerplate na UI** e **developer experience superior** em aplicações Flutter de qualquer complexidade! 🎯
