# Presentation Modules - Clean Architecture

## 📚 Visão Geral

Os **Modules** na camada de **Presentation** são responsáveis por **injeção de dependências** e **configuração de rotas** usando Flutter Modular. Existem dois padrões distintos: **Módulos Internos** para features da aplicação e **Módulos de Pacotes** para módulos externos com configuração flexível.

### 🎯 Princípios Fundamentais dos Modules

**O QUE os Modules FAZEM:**
- ✅ **Injetam Dependências**: Configuram DI container com controllers e UseCases
- ✅ **Definem Rotas**: Estruturam navegação e hierarquia de páginas
- ✅ **Importam Módulos**: Reutilizam dependências de módulos auxiliares
- ✅ **Exportam Dependências**: Disponibilizam services para outros módulos
- ✅ **Configuram Features**: Setup específico por contexto/funcionalidade

**O QUE os Modules NÃO FAZEM:**
- ❌ **Não contêm lógica de negócio**: Apenas configuração e DI
- ❌ **Não implementam regras**: Apenas orquestram dependências
- ❌ **Não fazem comunicação**: Apenas injetam quem faz
- ❌ **Não mantêm estado**: Apenas configuram quem mantém
- ❌ **Não fazem transformações**: Apenas conectam components

### 🏗️ Localização e Estrutura

```
lib/src/presentation/
├── modules/                         # Módulos internos da aplicação
│   ├── main_module.dart            # Módulo principal da app
│   ├── home_module.dart            # Módulo do home
│   ├── auth_module.dart            # Módulo de autenticação
│   └── feature_module.dart         # Módulos específicos por feature
└── [package]_module.dart           # Módulos de pacotes externos
    ├── funnel_module.dart          # Módulo do pacote funnel
    ├── candidates_module.dart      # Módulo do pacote candidates
    └── commission_module.dart      # Módulo do pacote commission
```

---

## 🔍 Elementos Essenciais dos Modules

### 🧩 1. Binds (Injeção de Dependências)

**Responsabilidade:** Configurar dependency injection de controllers e services da apresentação

```dart
@override
void binds(Injector i) {
  // Controllers da apresentação
  i.addLazySingleton<MainController>(MainController.new);
  i.add<FeatureController>(FeatureController.new);
  
  // Services específicos se necessário
  i.addInstance<SystemType>(SystemType.dc);
}
```

### 🛣️ 2. Routes (Configuração de Rotas)

**Responsabilidade:** Definir navegação e hierarquia de páginas do módulo

```dart
@override
void routes(RouteManager r) {
  r.child('/home/', child: (_) => const HomePage());
  r.module('/profile/', module: ProfileModule());
}
```

### 📦 3. Imports (Módulos Auxiliares)

**Responsabilidade:** Importar dependências de outros módulos fundamentais

```dart
@override
List<Module> get imports => [
  CoreModule(),      // Core utilities
  HttpModule(),      // HTTP client configuration
  LocalUserModule(), // User persistence
];
```

### 🔄 4. ExportedBinds (Dependências Exportadas)

**Responsabilidade:** Disponibilizar dependências para outros módulos

```dart
@override
void exportedBinds(Injector i) {
  // Dependências que outros módulos podem usar
  i.add<IFeatureUsecase>(FeatureUsecase.new);
  i.add<IFeatureRepository>(FeatureRepository.new);
}
```

---

## 📚 Exemplos Práticos Reais

### 1. Módulo Interno - MainModule

```dart
import 'package:cogna_resale_candidates/cogna_resale_candidates.dart'
    show CandidatesModule, CandidatesRoutes, EnrollmentDetailsPageArgs;
import 'package:cogna_resale_core/cogna_resale_core.dart';
import 'package:cogna_resale_funnel/cogna_resale_funnel.dart'
    show FunnelModule, FunnelRoutes;
import 'package:flutter_modular/flutter_modular.dart';

import '../../../app/presentation/controllers/app_controller.dart';
import '../controllers/main_controller.dart';
import '../pages/main/main_page.dart';
import '../routes/home_routes.dart';
import '../routes/main_routes.dart';
import 'home_module.dart';

/// Módulo principal da aplicação
/// 
/// Responsável por configurar a navegação principal,
/// integrar módulos de pacotes e coordenar controllers globais.
class MainModule extends Module {
  @override
  void binds(Injector i) {
    // Session controller com dependências do app
    i.addInstance<SessionController>(
      SessionController(
        DM.i.get<AppController>().session,
        userUsecase: DM.i.get<IUserUsecase>(),
        localUserUsecase: DM.i.get<ILocalUserUsecase>(),
      ),
    );
    
    // Controller principal
    i.addLazySingleton<MainController>(MainController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      MainRoutes.root.path,
      child: (_) => const MainPage(),
      children: [
        // Módulos internos
        ModuleRoute(HomeRoutes.root.path, module: HomeModule()),
        ModuleRoute(ProfileRoutes.root.path, module: ProfileModule()),
        ModuleRoute(HelpCenterRoutes.root.path, module: HelpCenterModule()),
        ModuleRoute(NotificationsRoutes.root.path, module: NotificationsModule()),
        
        // Módulos de pacotes com configuração
        ModuleRoute(
          FunnelRoutes.i(parentRoot: MainRoutes.root.completePath).rootPath,
          module: FunnelModule(
            system: SystemType.dc,
            goToEnrollmentDetails: (idOrigin) {
              Nav.to.popUntilAndPushNamed(MainRoutes.root, CandidatesRoutes.root);
              if (idOrigin.isEmpty) return;
              Nav.to.pushNamed(
                CandidatesRoutes.enrollmentDetails,
                arguments: EnrollmentDetailsPageArgs(idOrigin: idOrigin),
              );
            },
          ),
        ),
        ModuleRoute(
          CandidatesRoutes.i(parentRoot: MainRoutes.root.completePath).rootPath,
          module: CandidatesModule(),
        ),
      ],
    );
  }
}
```

**Características:**
- **Binds específicos**: Controllers globais da aplicação
- **Roteamento complexo**: Integração de múltiplos módulos
- **Callback injection**: Coordenação entre módulos de pacotes
- **Configuração de pacotes**: Inicialização com parâmetros

### 2. Módulo de Pacote - FunnelModule

```dart
import 'package:cogna_offer_utils/cogna_offer_utils.dart';
import 'package:cogna_resale_core/cogna_resale_core.dart';
import 'package:flutter_modular/flutter_modular.dart';

import '../data/datasources/enrollment_datasource.dart';
import '../domain/repositories/i_enrollment_repository.dart';
import '../domain/usecases/i_enrollment_usecase.dart';
import '../infra/repositories/enrollment_repository.dart';
import '../infra/usecases/enrollment_usecase.dart';
import 'controllers/enrollment_controller.dart';
import 'funnel_routes.dart';
import 'pages/offers/offers_page.dart';

/// Módulo do pacote Cogna Resale Funnel
/// 
/// Permite configuração flexível através de parâmetros
/// e coordenação com outros módulos da aplicação host.
class FunnelModule extends Module {
  FunnelModule({
    required this.system,
    this.goToEnrollmentDetails,
  });

  final SystemType system;
  final Function(String idOrigin)? goToEnrollmentDetails;

  @override
  List<Module> get imports => [HttpModule()];

  @override
  void binds(Injector i) {
    /// Configuração do sistema
    i.addInstance<SystemType>(system);

    /// Camada Data
    i.add<IEnrollmentDatasource>(EnrollmentDatasource.new);
    
    /// Camada Infrastructure
    i.add<IEnrollmentRepository>(EnrollmentRepository.new);
    i.add<IEnrollmentUsecase>(EnrollmentUsecase.new);

    /// Session da aplicação host
    i.add<SessionEntity>(() => DM.i.get<SessionController>().state);

    /// Controllers da apresentação
    i.add<EnrollmentController>(EnrollmentController.new);
    i.add<OffersController>(OffersController.new);
    i.add<VoucherController>(VoucherController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      transition: TransitionType.fadeIn,
      child: (_) => OffersFiltersPage(
        args: r.args.data is OffersFiltersPageArgs
            ? r.args.data
            : OffersFiltersPageArgs(
                onSearch: (_) {},
                filters: OfferFiltersModel.fromMap({}),
              ),
      ),
    );
    
    r.child(
      FunnelRoutes.offers.path,
      child: (_) => OffersPage(
        args: r.args.data is OffersPageArgs
            ? r.args.data
            : OffersPageArgs(
                filters: OfferFiltersModel.fromMap({}),
                onChanged: (_) {},
              ),
      ),
    );
    
    r.child(
      FunnelRoutes.createEnrollment.path,
      child: (_) => CreateEnrollmentPage(
        goToEnrollmentDetails: goToEnrollmentDetails,
        args: r.args.data is CreateEnrollmentPageArgs
            ? r.args.data
            : CreateEnrollmentPageArgs(
                isEasyPay: false,
                easyPayAvailable: false,
                offer: OfferModel.fromMap({}),
                shiftType: OfferShiftType.virtual,
                offerPricing: OfferPricingModel.fromMap({}),
              ),
      ),
    );
  }
}
```

**Características:**
- **Configuração flexível**: Parâmetros no construtor
- **Imports específicos**: HttpModule para comunicação
- **DI completo**: Todas as camadas da Clean Architecture
- **Callback injection**: goToEnrollmentDetails para coordenação
- **Args default**: Tratamento de argumentos opcionais

### 3. Módulo com ExportedBinds - AuthModule

```dart
import 'package:base_core/base_core.dart';
import 'package:cogna_resale_core/cogna_resale_core.dart';
import 'package:flutter_modular/flutter_modular.dart';

import '../../data/datasources/auth_datasource.dart';
import '../../domain/repositories/i_auth_repository.dart';
import '../../domain/usecases/i_auth_usecase.dart';
import '../../infra/repositories/auth_repository.dart';
import '../../infra/usecases/auth_usecase.dart';
import '../controllers/auth_controller.dart';
import '../routes/auth_routes.dart';
import 'login_module.dart';
import 'register_module.dart';

/// Módulo de autenticação da aplicação
/// 
/// Responsável por configurar todo o fluxo de auth,
/// exportando dependências para outros módulos.
class AuthModule extends Module {
  AuthModule({
    this.onLogin,
    this.redirectToAfterLogin,
    this.system = SystemType.ce,
  });

  final SystemType system;
  final BasePath? redirectToAfterLogin;
  final Future<Either<ICustomFailure, SessionEntity>> Function(SessionEntity)? onLogin;

  @override
  List<Module> get imports => [CoreModule(), LocalUserModule()];

  @override
  void exportedBinds(Injector i) {
    /// HTTP client configurado
    i.addInstance<Dio>(
      Dio(
        BaseOptions(
          baseUrl: AppConfiguration.environment.appBaseUrl,
          sendTimeout: const Duration(milliseconds: 60000),
          connectTimeout: const Duration(milliseconds: 60000),
          receiveTimeout: const Duration(milliseconds: 60000),
        ),
      )..interceptors.addAll([
        if (kDebugMode)
          LogInterceptor(
            requestBody: true,
            responseBody: true,
            logPrint: (value) => log('$value'),
          ),
      ]),
    );

    /// HTTP driver
    i.add<IHttpDriver>(DioClientDriver.new);

    /// Dependências de autenticação (exportadas)
    i.add<IAuthDatasource>(AuthDatasource.new);
    i.add<IAuthRepository>(AuthRepository.new);
    i.add<IAuthUsecase>(AuthUsecase.new);

    /// Controllers exportados
    i.add<AuthController>(AuthController.new);
  }

  @override
  void binds(Injector i) {
    /// Controllers locais
    i.addLazySingleton<UserLocalPreferencesController>(
      UserLocalPreferencesController.new,
    );
    i.add<BiometricController>(BiometricController.new);

    /// Configuração do sistema
    i.addInstance<SystemType>(system);
  }

  @override
  void routes(RouteManager r) {
    r.redirect(AuthRoutes.root.path, to: LoginRoutes.root.completePath);
    
    r.module(
      LoginRoutes.root.path,
      module: LoginModule(
        redirectToAfterLogin: redirectToAfterLogin,
        onLogin: onLogin,
      ),
    );
    
    r.module(
      RegisterRoutes.root.path,
      module: RegisterModule(
        redirectToAfterLogin: redirectToAfterLogin,
        onLogin: onLogin,
      ),
    );
    
    r.module(RecoveryAccountRoutes.root.path, module: RecoveryAccountModule());
  }
}
```

**Características:**
- **ExportedBinds**: Dependências disponíveis para outros módulos
- **Imports múltiplos**: CoreModule, LocalUserModule
- **Configuração avançada**: HTTP client com interceptors
- **Callback propagation**: onLogin propagado para submódulos
- **Redirects**: Configuração de redirecionamento de rotas

---

## 🎨 Padrões de Implementação

### 1. Módulo Interno Simples

```dart
import 'package:flutter_modular/flutter_modular.dart';

import '../controllers/feature_controller.dart';
import '../pages/feature_page.dart';
import '../routes/feature_routes.dart';

/// Módulo da feature específica
/// 
/// Responsável por configurar controllers e páginas
/// da funcionalidade específica.
class FeatureModule extends Module {
  @override
  void binds(Injector i) {
    // Controllers específicos da feature
    i.add<FeatureController>(FeatureController.new);
    i.add<FeatureListController>(FeatureListController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      child: (_) => const FeaturePage(),
    );
    
    r.child(
      FeatureRoutes.list.path,
      child: (_) => FeatureListPage(
        args: r.args.data as FeatureListPageArgs?,
      ),
    );
    
    r.child(
      FeatureRoutes.details.path,
      child: (_) => FeatureDetailsPage(
        args: r.args.data as FeatureDetailsPageArgs,
      ),
    );
  }
}
```

### 2. Módulo com Dependências Externas

```dart
import 'package:flutter_modular/flutter_modular.dart';

import '../../data/datasources/feature_datasource.dart';
import '../../domain/repositories/i_feature_repository.dart';
import '../../domain/usecases/i_feature_usecase.dart';
import '../../infra/repositories/feature_repository.dart';
import '../../infra/usecases/feature_usecase.dart';
import '../controllers/feature_controller.dart';

/// Módulo com dependências completas da Clean Architecture
class FullFeatureModule extends Module {
  @override
  List<Module> get imports => [CoreModule(), HttpModule()];

  @override
  void binds(Injector i) {
    /// Data Layer
    i.add<IFeatureDatasource>(FeatureDatasource.new);
    
    /// Infrastructure Layer
    i.add<IFeatureRepository>(FeatureRepository.new);
    i.add<IFeatureUsecase>(FeatureUsecase.new);
    
    /// Presentation Layer
    i.add<FeatureController>(FeatureController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      child: (_) => const FeaturePage(),
    );
  }
}
```

### 3. Módulo de Pacote Configurável

```dart
import 'package:flutter_modular/flutter_modular.dart';

/// Módulo de pacote com configuração flexível
class PackageModule extends Module {
  PackageModule({
    required this.config,
    this.onEvent,
    this.enableFeatures = const [],
  });

  final PackageConfig config;
  final Function(PackageEvent)? onEvent;
  final List<FeatureType> enableFeatures;

  @override
  List<Module> get imports => [
    CoreModule(),
    if (config.requiresHttp) HttpModule(),
    if (config.requiresStorage) StorageModule(),
  ];

  @override
  void exportedBinds(Injector i) {
    // Configuração exportada para outros módulos
    i.addInstance<PackageConfig>(config);
    
    // Services exportados baseados na configuração
    if (enableFeatures.contains(FeatureType.analytics)) {
      i.add<IAnalyticsService>(AnalyticsService.new);
    }
    
    if (enableFeatures.contains(FeatureType.notifications)) {
      i.add<INotificationService>(NotificationService.new);
    }
  }

  @override
  void binds(Injector i) {
    // Callback para eventos
    i.addInstance<Function(PackageEvent)?>(onEvent);
    
    // Controllers baseados em features habilitadas
    if (enableFeatures.contains(FeatureType.userManagement)) {
      i.add<UserManagementController>(UserManagementController.new);
    }
    
    if (enableFeatures.contains(FeatureType.contentManagement)) {
      i.add<ContentManagementController>(ContentManagementController.new);
    }
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      child: (_) => PackageHomePage(config: config),
    );

    // Rotas condicionais baseadas em features
    if (enableFeatures.contains(FeatureType.userManagement)) {
      r.child('/users/', child: (_) => const UsersPage());
    }
    
    if (enableFeatures.contains(FeatureType.contentManagement)) {
      r.child('/content/', child: (_) => const ContentPage());
    }
  }
}
```

### 4. Módulo com Validação e Setup

```dart
import 'package:flutter_modular/flutter_modular.dart';

/// Módulo com validação de configuração obrigatória
class ValidatedModule extends Module {
  ValidatedModule({
    required this.apiKey,
    required this.environment,
    this.enableLogging = false,
  }) : assert(apiKey.isNotEmpty, 'API Key is required'),
       assert(environment != Environment.unknown, 'Valid environment required');

  final String apiKey;
  final Environment environment;
  final bool enableLogging;

  @override
  List<Module> get imports => [
    CoreModule(),
    HttpModule(),
    if (enableLogging) LoggingModule(),
  ];

  @override
  void exportedBinds(Injector i) {
    // HTTP client configurado com API key
    i.addInstance<ApiConfig>(
      ApiConfig(
        apiKey: apiKey,
        environment: environment,
        enableLogging: enableLogging,
      ),
    );
    
    // HTTP client customizado
    i.add<IApiClient>(() => ApiClient(
      config: i.get<ApiConfig>(),
    ));
  }

  @override
  void binds(Injector i) {
    // Services específicos do ambiente
    switch (environment) {
      case Environment.development:
        i.add<IDataService>(MockDataService.new);
        break;
      case Environment.staging:
      case Environment.production:
        i.add<IDataService>(ApiDataService.new);
        break;
      case Environment.unknown:
        throw Exception('Invalid environment');
    }
    
    // Controller principal
    i.add<MainController>(MainController.new);
  }

  @override
  void routes(RouteManager r) {
    // Rota inicial baseada no ambiente
    final initialPage = environment == Environment.development
        ? const DevelopmentHomePage()
        : const ProductionHomePage();
        
    r.child(
      Nav.to.initialRoute,
      child: (_) => initialPage,
    );
    
    // Rotas de debug apenas em desenvolvimento
    if (environment == Environment.development) {
      r.child('/debug/', child: (_) => const DebugPage());
    }
  }
}
```

---

## 📋 Templates para Modules

### Template - Módulo Interno

```dart
import 'package:flutter_modular/flutter_modular.dart';

import '../controllers/[feature]_controller.dart';
import '../pages/[feature]_page.dart';
import '../routes/[feature]_routes.dart';

/// Módulo da feature [FeatureName]
/// 
/// Responsável por [descrição da responsabilidade]
/// e configuração das páginas de [contexto].
class [FeatureName]Module extends Module {
  @override
  void binds(Injector i) {
    // Controllers específicos da feature
    i.add<[FeatureName]Controller>([FeatureName]Controller.new);
    // Adicionar outros controllers necessários
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      child: (_) => const [FeatureName]Page(),
    );
    
    // Outras rotas da feature
    r.child(
      [FeatureName]Routes.details.path,
      child: (_) => [FeatureName]DetailsPage(
        args: r.args.data as [FeatureName]DetailsPageArgs,
      ),
    );
  }
}
```

### Template - Módulo de Pacote

```dart
import 'package:flutter_modular/flutter_modular.dart';

import '../../data/datasources/[feature]_datasource.dart';
import '../../domain/repositories/i_[feature]_repository.dart';
import '../../domain/usecases/i_[feature]_usecase.dart';
import '../../infra/repositories/[feature]_repository.dart';
import '../../infra/usecases/[feature]_usecase.dart';
import '../controllers/[feature]_controller.dart';
import '../pages/[feature]_page.dart';
import '[feature]_routes.dart';

/// Módulo do pacote [PackageName]
/// 
/// [Descrição do pacote e sua funcionalidade]
/// Permite configuração flexível através de parâmetros.
class [PackageName]Module extends Module {
  [PackageName]Module({
    required this.config,
    this.onEvent,
    this.additionalParams,
  });

  final [PackageName]Config config;
  final Function([PackageName]Event)? onEvent;
  final [AdditionalParamsType]? additionalParams;

  @override
  List<Module> get imports => [
    CoreModule(),
    HttpModule(),
    // Outros módulos necessários
  ];

  @override
  void exportedBinds(Injector i) {
    /// Configuração exportada
    i.addInstance<[PackageName]Config>(config);
    
    /// Dependências exportadas para outros módulos
    i.add<I[Feature]Usecase>([Feature]Usecase.new);
    i.add<I[Feature]Repository>([Feature]Repository.new);
  }

  @override
  void binds(Injector i) {
    /// Callback para eventos
    i.addInstance<Function([PackageName]Event)?>(onEvent);
    
    /// Parâmetros adicionais
    if (additionalParams != null) {
      i.addInstance<[AdditionalParamsType]>(additionalParams!);
    }
    
    /// Data Layer
    i.add<I[Feature]Datasource>([Feature]Datasource.new);
    
    /// Infrastructure Layer
    i.add<I[Feature]Repository>([Feature]Repository.new);
    i.add<I[Feature]Usecase>([Feature]Usecase.new);
    
    /// Presentation Layer
    i.add<[Feature]Controller>([Feature]Controller.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      Nav.to.initialRoute,
      child: (_) => [PackageName]HomePage(config: config),
    );
    
    // Outras rotas do pacote
    r.child(
      [PackageName]Routes.feature.path,
      child: (_) => [Feature]Page(
        args: r.args.data as [Feature]PageArgs?,
        onEvent: onEvent,
      ),
    );
  }
}
```

### Convenções de Modules

**Nomenclatura:**
- Modules: `[FeatureName]Module` (sempre sufixo Module)
- Configuração: `[PackageName]Config` para parâmetros
- Callbacks: `Function([Event])? onEvent` para coordenação

**Estrutura:**
- Imports: Módulos auxiliares necessários
- ExportedBinds: Dependências para outros módulos
- Binds: Dependências locais do módulo
- Routes: Definição das páginas e navegação

**Organização:**
- Um módulo por feature/pacote
- Separação clara entre binds e exportedBinds
- Validação de parâmetros obrigatórios
- Configuração condicional baseada em parâmetros

---

## 📋 Checklist para Modules

### Checklist de Criação ✅

**Estrutura do Module:**
- [ ] Escolha do tipo correto: Interno vs Pacote
- [ ] Nomenclatura seguindo padrão: `[Name]Module`
- [ ] Imports de módulos auxiliares necessários
- [ ] Documentação clara da responsabilidade

**Binds (Dependências Locais):**
- [ ] Controllers da apresentação configurados
- [ ] UseCases injetados corretamente
- [ ] Repositories configurados
- [ ] DataSources implementados
- [ ] Configurações específicas (SystemType, etc.)

**ExportedBinds (Dependências Exportadas):**
- [ ] Services exportados para outros módulos
- [ ] HTTP client configurado (se necessário)
- [ ] Dependências compartilhadas disponibilizadas
- [ ] Configurações globais exportadas

**Routes (Configuração de Rotas):**
- [ ] Página inicial definida
- [ ] Sub-rotas configuradas
- [ ] Módulos filhos integrados
- [ ] Arguments handling implementado
- [ ] Transitions configuradas apropriadamente

**Configuração de Pacotes:**
- [ ] Parâmetros de configuração validados
- [ ] Callbacks para coordenação implementados
- [ ] Features condicionais baseadas em config
- [ ] Defaults apropriados para args opcionais

**Imports e Dependências:**
- [ ] CoreModule importado (se necessário)
- [ ] HttpModule importado para comunicação
- [ ] Outros módulos auxiliares importados
- [ ] Dependências circulares evitadas

---

## 🎯 Diretrizes para Modules

### ✅ Boas Práticas

```dart
// ✅ Configuração clara de módulo de pacote
class PaymentModule extends Module {
  PaymentModule({
    required this.apiKey,
    required this.environment,
    this.onPaymentComplete,
  }) : assert(apiKey.isNotEmpty, 'API Key is required');

  final String apiKey;
  final Environment environment;
  final Function(PaymentResult)? onPaymentComplete;
}

// ✅ Binds organizados por camada
@override
void binds(Injector i) {
  /// Configuration
  i.addInstance<PaymentConfig>(PaymentConfig(
    apiKey: apiKey,
    environment: environment,
  ));
  
  /// Data Layer
  i.add<IPaymentDatasource>(PaymentDatasource.new);
  
  /// Infrastructure Layer
  i.add<IPaymentRepository>(PaymentRepository.new);
  i.add<IPaymentUsecase>(PaymentUsecase.new);
  
  /// Presentation Layer
  i.add<PaymentController>(PaymentController.new);
}

// ✅ Routes com args handling
r.child(
  PaymentRoutes.checkout.path,
  child: (_) => CheckoutPage(
    args: r.args.data is CheckoutPageArgs
        ? r.args.data
        : CheckoutPageArgs.empty(),
    onComplete: onPaymentComplete,
  ),
);

// ✅ ExportedBinds bem definidos
@override
void exportedBinds(Injector i) {
  // HTTP client configurado
  i.addInstance<Dio>(createHttpClient());
  
  // Services exportados
  i.add<IPaymentService>(PaymentService.new);
  i.add<IPaymentValidator>(PaymentValidator.new);
}

// ✅ Imports necessários
@override
List<Module> get imports => [
  CoreModule(),
  HttpModule(),
  if (environment.requiresLogging) LoggingModule(),
];
```

### ❌ Evitar

```dart
// ❌ Módulo sem validação de parâmetros
class BadModule extends Module {
  BadModule({this.config}); // ❌ Sem required nem validação
  
  final BadConfig? config;
}

// ❌ Binds desorganizados
@override
void binds(Injector i) {
  i.add<Controller1>(Controller1.new);
  i.add<IRepository1>(Repository1.new); // ❌ Misturado com controller
  i.add<Controller2>(Controller2.new);
  i.add<IDatasource1>(Datasource1.new); // ❌ Sem organização por camada
}

// ❌ Routes sem tratamento de args
r.child(
  '/checkout/',
  child: (_) => CheckoutPage(
    args: r.args.data, // ❌ Cast direto sem verificação
  ),
);

// ❌ ExportedBinds misturado com binds locais
@override
void exportedBinds(Injector i) {
  i.add<IPublicService>(PublicService.new); // ✅ Export
  i.add<InternalController>(InternalController.new); // ❌ Não deveria exportar
}

// ❌ Imports desnecessários
@override
List<Module> get imports => [
  CoreModule(),
  HttpModule(), // ❌ Se não usa HTTP
  DatabaseModule(), // ❌ Se não usa database
  AllPossibleModules(), // ❌ Import excessivo
];

// ❌ Não usar assert para validação
class UnsafeModule extends Module {
  UnsafeModule({this.apiKey});
  
  final String? apiKey; // ❌ Deveria ser required com assert
  
  @override
  void binds(Injector i) {
    // ❌ Usa apiKey sem validar se não é null
    i.addInstance<ApiConfig>(ApiConfig(apiKey: apiKey!));
  }
}
```

---

## 🚀 Uso dos Modules

### 1. Configuração no App Principal

```dart
// No main.dart
void main() {
  runApp(ModularApp(
    module: AppModule(),
    child: const MyApp(),
  ));
}

// App Module principal
class AppModule extends Module {
  @override
  List<Module> get imports => [CoreModule()];

  @override
  void routes(RouteManager r) {
    r.module('/auth/', module: AuthModule(
      system: SystemType.dc,
      onLogin: (session) async {
        // Lógica de login
        return Right(session);
      },
    ));
    
    r.module('/main/', module: MainModule());
  }
}
```

### 2. Integração de Módulos de Pacotes

```dart
// Configuração de múltiplos pacotes
class MainModule extends Module {
  @override
  void routes(RouteManager r) {
    r.child('/home/', child: (_) => const HomePage(), children: [
      // Pacote de vendas
      ModuleRoute(
        FunnelRoutes.i(parentRoot: '/main/home/').rootPath,
        module: FunnelModule(
          system: SystemType.dc,
          goToEnrollmentDetails: _handleEnrollmentNavigation,
        ),
      ),
      
      // Pacote de candidatos
      ModuleRoute(
        CandidatesRoutes.i(parentRoot: '/main/home/').rootPath,
        module: CandidatesModule(
          enableAnalytics: true,
          onCandidateUpdated: _handleCandidateUpdate,
        ),
      ),
      
      // Pacote de comissões
      ModuleRoute(
        CommissionRoutes.i(parentRoot: '/main/home/').rootPath,
        module: CommissionModule(
          apiVersion: 'v2',
          onCommissionCalculated: _handleCommissionCalculation,
        ),
      ),
    ]);
  }
  
  void _handleEnrollmentNavigation(String idOrigin) {
    // Lógica de navegação entre pacotes
  }
  
  void _handleCandidateUpdate(CandidateEntity candidate) {
    // Lógica de atualização de candidato
  }
  
  void _handleCommissionCalculation(CommissionResult result) {
    // Lógica de comissão calculada
  }
}
```

### 3. Módulo com Configuração Condicional

```dart
class EnvironmentAwareModule extends Module {
  @override
  void binds(Injector i) {
    // Configuração baseada em ambiente
    if (kDebugMode) {
      i.add<IAnalyticsService>(MockAnalyticsService.new);
      i.add<ILogService>(VerboseLogService.new);
    } else {
      i.add<IAnalyticsService>(FirebaseAnalyticsService.new);
      i.add<ILogService>(ProductionLogService.new);
    }
    
    // Controllers sempre necessários
    i.add<HomeController>(HomeController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child('/home/', child: (_) => const HomePage());
    
    // Rotas de debug apenas em desenvolvimento
    if (kDebugMode) {
      r.child('/debug/', child: (_) => const DebugPage());
      r.child('/logs/', child: (_) => const LogsPage());
    }
  }
}
```

---

## 🎯 Padrões Avançados

### 1. Módulo com Feature Flags

```dart
class FeatureFlagModule extends Module {
  FeatureFlagModule({required this.features});

  final Set<FeatureFlag> features;

  @override
  void binds(Injector i) {
    // Feature flags instance
    i.addInstance<Set<FeatureFlag>>(features);
    
    // Controllers baseados em features
    if (features.contains(FeatureFlag.advancedSearch)) {
      i.add<AdvancedSearchController>(AdvancedSearchController.new);
    }
    
    if (features.contains(FeatureFlag.realTimeNotifications)) {
      i.add<RealTimeNotificationController>(RealTimeNotificationController.new);
    }
    
    if (features.contains(FeatureFlag.darkMode)) {
      i.add<ThemeController>(ThemeController.new);
    }
  }

  @override
  void routes(RouteManager r) {
    r.child('/home/', child: (_) => const HomePage());
    
    // Rotas condicionais
    if (features.contains(FeatureFlag.advancedSearch)) {
      r.child('/search/', child: (_) => const AdvancedSearchPage());
    }
    
    if (features.contains(FeatureFlag.userDashboard)) {
      r.child('/dashboard/', child: (_) => const UserDashboardPage());
    }
  }
}
```

### 2. Módulo com Lazy Loading

```dart
class LazyLoadingModule extends Module {
  @override
  void binds(Injector i) {
    // Lazy singletons para performance
    i.addLazySingleton<ExpensiveService>(ExpensiveService.new);
    i.addLazySingleton<HeavyController>(HeavyController.new);
    
    // Services que só são carregados quando necessários
    i.add<ILazyDataService>(() async {
      await Future.delayed(const Duration(milliseconds: 100));
      return LazyDataService();
    });
  }
}
```

### 3. Módulo com Interceptors

```dart
class InterceptorModule extends Module {
  @override
  void exportedBinds(Injector i) {
    i.addInstance<Dio>(
      Dio()..interceptors.addAll([
        AuthInterceptor(),
        ErrorHandlingInterceptor(),
        LoggingInterceptor(),
        RetryInterceptor(retries: 3),
        CacheInterceptor(),
      ]),
    );
  }
}
```

---

## 🎯 Resumo dos Benefícios

### ✅ **Injeção de Dependências Organizada**
- **Separação clara**: Binds locais vs exportedBinds
- **Camadas bem definidas**: Data, Infrastructure, Presentation
- **Configuração flexível**: Parâmetros e validação
- **Lifecycle management**: Singleton, lazy loading, instances

### ✅ **Roteamento Estruturado**
- **Hierarquia clara**: Módulos principais e submódulos
- **Args handling**: Tratamento seguro de argumentos
- **Navegação coordenada**: Callbacks entre módulos
- **Transições configuráveis**: Animações e transições

### ✅ **Modularidade e Reutilização**
- **Módulos de pacotes**: Configuração flexível e reutilização
- **Imports organizados**: Dependências claras e controladas
- **Feature flags**: Ativação condicional de funcionalidades
- **Environment aware**: Configuração baseada em ambiente

### ✅ **Maintainability**
- **Templates consistentes**: Estrutura padrão para novos módulos
- **Validação rigorosa**: Assert statements e type safety
- **Documentação clara**: Responsabilidades bem definidas
- **Testabilidade**: DI facilita mocking e testes

Esta arquitetura de Modules garante **injeção de dependências robusta**, **roteamento organizado** e **modularidade escalável** em aplicações Flutter complexas! 🎯