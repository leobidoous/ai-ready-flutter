# Presentation Routes - Clean Architecture

## 📚 Visão Geral

As **Routes** na camada de **Presentation** são responsáveis por **definir navegação** e **organizar hierarquia de rotas** da aplicação. Existem dois padrões distintos: **Rotas Internas** para módulos da aplicação e **Rotas de Pacotes** para módulos importados que precisam de configuração flexível.

### 🎯 Princípios Fundamentais das Routes

**O QUE as Routes FAZEM:**
- ✅ **Definem Hierarquia**: Estruturam navegação em árvore de rotas
- ✅ **Organizam Módulos**: Separam rotas por contexto/feature
- ✅ **Facilitam Navegação**: Fornecem paths tipificados e seguros
- ✅ **Integram Pacotes**: Permitem configuração flexível de módulos externos
- ✅ **Mantêm Consistência**: Padrões uniformes de nomenclatura e estrutura

**O QUE as Routes NÃO FAZEM:**
- ❌ **Não contêm lógica de negócio**: Apenas definem paths de navegação
- ❌ **Não fazem validação**: Apenas estruturam rotas, não validam acesso
- ❌ **Não gerenciam estado**: Apenas definem caminhos, não mantêm estado
- ❌ **Não implementam navegação**: Apenas fornecem paths para Nav.to
- ❌ **Não fazem autenticação**: Apenas estruturam, não controlam acesso

### 🏗️ Localização e Estrutura

```
lib/src/presentation/
├── routes/                          # Rotas internas da aplicação
│   ├── main_routes.dart            # Rotas principais
│   ├── home_routes.dart            # Rotas do módulo home
│   ├── auth_routes.dart            # Rotas de autenticação
│   └── feature_routes.dart         # Rotas específicas por feature
└── [package]_routes.dart           # Rotas de pacotes externos
    ├── funnel_routes.dart          # Rota do pacote funnel
    ├── candidates_routes.dart      # Rota do pacote candidates
    └── commission_routes.dart      # Rota do pacote commission
```

---

## 🔍 Tipos de Routes

### 🏠 1. Rotas Internas (Aplicação)

**Características:**
- ✅ **Estrutura simples**: Apenas `static BasePath`
- ✅ **Integração direta**: Referenciam outras rotas da mesma aplicação
- ✅ **Sem configuração**: Instanciação automática
- ✅ **Performance**: Zero overhead de inicialização

**Quando usar:**
- Módulos internos da aplicação
- Rotas que não precisam de configuração externa
- Hierarquias fixas e conhecidas em tempo de compilação

### 📦 2. Rotas de Pacotes/Módulos

**Características:**
- ✅ **Singleton pattern**: Instância única controlada
- ✅ **Parent root configurável**: Flexibilidade de integração
- ✅ **Inicialização obrigatória**: Setup via método `.i()`
- ✅ **Reutilização**: Pode ser usado em múltiplas aplicações

**Quando usar:**
- Pacotes/módulos externos
- Rotas que precisam ser integradas flexivelmente
- Módulos que podem ter diferentes parent roots
- Bibliotecas que serão reutilizadas

---

## 📚 Exemplos Práticos Reais

### 1. Rotas Internas - HomeRoutes

```dart
import 'package:base_core/base_core.dart';

import 'main_routes.dart';

/// Rotas do módulo Home da aplicação
/// 
/// Define a navegação interna do módulo home,
/// referenciando diretamente as rotas principais.
class HomeRoutes {
  // Rota base do módulo home
  static BasePath root = BasePath('/home/', MainRoutes.root);
  
  // Sub-rotas do módulo (exemplos)
  static BasePath dashboard = BasePath('/dashboard/', root);
  static BasePath profile = BasePath('/profile/', root);
  static BasePath settings = BasePath('/settings/', root);
}
```

**Características:**
- **Estrutura simples**: Apenas campos `static BasePath`
- **Dependência direta**: Referencia `MainRoutes.root` diretamente
- **Zero configuração**: Pronto para uso imediato
- **Hierarquia clara**: `/main/start/home/dashboard/`

### 2. Rotas de Pacotes - FunnelRoutes

```dart
import 'package:base_core/base_core.dart';

/// Rotas do pacote Cogna Resale Funnel
/// 
/// Permite integração flexível em diferentes aplicações
/// através de configuração do parent root.
class FunnelRoutes {
  // Private constructor para singleton
  FunnelRoutes._internal({String parentRoot = ''});

  /// Método público para inicializar e/ou obter a instância
  /// 
  /// [parentRoot] define onde o módulo será integrado
  /// na hierarquia de rotas da aplicação host.
  static FunnelRoutes i({String parentRoot = ''}) {
    _parentRoot = parentRoot;
    // Inicializa _instance se for null
    _instance ??= FunnelRoutes._internal(parentRoot: parentRoot);
    return _instance!;
  }

  // Instância singleton privada
  static FunnelRoutes? _instance;
  static late String? _parentRoot;

  // Getters de conveniência
  String get rootPath => root.path;
  String get rootCompletePath => root.completePath;

  // Rota base do módulo funnel
  static final BasePath root = BasePath(
    '/resale_funnel/',
    _parentRoot != null ? BasePath(_parentRoot!) : null,
  );
  
  // Sub-rotas do módulo
  static final BasePath offers = BasePath('/offers/', root);
  static final BasePath createEnrollment = BasePath('/create_enrollment/', root);
  static final BasePath enrollmentDetails = BasePath('/enrollment_details/', root);
}
```

**Características:**
- **Singleton pattern**: Controle de instância única
- **Parent root configurável**: Flexibilidade de integração
- **Inicialização obrigatória**: `FunnelRoutes.i(parentRoot: '/main/')`
- **Getters úteis**: `rootPath`, `rootCompletePath`

---

## 🎨 Padrões de Implementação

### 1. Rotas Internas Simples

```dart
import 'package:base_core/base_core.dart';

import 'main_routes.dart';

/// Rotas do módulo de autenticação
class AuthRoutes {
  static BasePath root = BasePath('/auth/', MainRoutes.root);
  static BasePath login = BasePath('/login/', root);
  static BasePath register = BasePath('/register/', root);
  static BasePath forgotPassword = BasePath('/forgot_password/', root);
  static BasePath resetPassword = BasePath('/reset_password/', root);
}
```

### 2. Rotas Internas com Hierarquia Complexa

```dart
import 'package:base_core/base_core.dart';

import 'main_routes.dart';

/// Rotas do módulo de e-commerce
class EcommerceRoutes {
  // Raiz do módulo
  static BasePath root = BasePath('/ecommerce/', MainRoutes.root);
  
  // Produtos
  static BasePath products = BasePath('/products/', root);
  static BasePath productDetails = BasePath('/details/', products);
  static BasePath productReviews = BasePath('/reviews/', products);
  
  // Carrinho
  static BasePath cart = BasePath('/cart/', root);
  static BasePath checkout = BasePath('/checkout/', cart);
  static BasePath payment = BasePath('/payment/', checkout);
  
  // Pedidos
  static BasePath orders = BasePath('/orders/', root);
  static BasePath orderDetails = BasePath('/details/', orders);
  static BasePath orderTracking = BasePath('/tracking/', orders);
}
```

### 3. Rotas de Pacote com Configuração Avançada

```dart
import 'package:base_core/base_core.dart';

/// Rotas do pacote Cogna Resale Candidates
/// 
/// Módulo para gerenciamento de candidatos com
/// configuração flexível de integração.
class CandidatesRoutes {
  CandidatesRoutes._internal({
    String parentRoot = '',
    this.enableAnalytics = true,
  });

  static CandidatesRoutes i({
    String parentRoot = '',
    bool enableAnalytics = true,
  }) {
    _parentRoot = parentRoot;
    _enableAnalytics = enableAnalytics;
    
    _instance ??= CandidatesRoutes._internal(
      parentRoot: parentRoot,
      enableAnalytics: enableAnalytics,
    );
    return _instance!;
  }

  static CandidatesRoutes? _instance;
  static late String? _parentRoot;
  static late bool _enableAnalytics;

  final bool enableAnalytics;

  // Getters de conveniência
  String get rootPath => root.path;
  String get rootCompletePath => root.completePath;
  bool get isAnalyticsEnabled => enableAnalytics;

  // Rotas do módulo
  static final BasePath root = BasePath(
    '/candidates/',
    _parentRoot != null ? BasePath(_parentRoot!) : null,
  );
  
  static final BasePath list = BasePath('/list/', root);
  static final BasePath details = BasePath('/details/', root);
  static final BasePath enrollment = BasePath('/enrollment/', root);
  static final BasePath enrollmentDetails = BasePath('/details/', enrollment);
  
  // Métodos utilitários
  static String buildEnrollmentDetailsPath(String enrollmentId) {
    return '${enrollmentDetails.completePath}?id=$enrollmentId';
  }
  
  static String buildCandidateDetailsPath(String candidateId) {
    return '${details.completePath}?id=$candidateId';
  }
}
```

### 4. Rotas de Pacote com Validação

```dart
import 'package:base_core/base_core.dart';

/// Rotas do pacote Cogna Resale Commission
/// 
/// Módulo de comissões com validação de configuração
/// e inicialização segura.
class CommissionRoutes {
  CommissionRoutes._internal({
    required String parentRoot,
    required this.apiVersion,
  }) : assert(parentRoot.isNotEmpty, 'Parent root cannot be empty');

  static CommissionRoutes i({
    required String parentRoot,
    String apiVersion = 'v1',
  }) {
    assert(parentRoot.isNotEmpty, 'Parent root is required');
    
    _parentRoot = parentRoot;
    _apiVersion = apiVersion;
    
    _instance ??= CommissionRoutes._internal(
      parentRoot: parentRoot,
      apiVersion: apiVersion,
    );
    return _instance!;
  }

  static CommissionRoutes? _instance;
  static late String _parentRoot;
  static late String _apiVersion;

  final String apiVersion;

  // Getters com validação
  String get rootPath {
    assert(_instance != null, 'CommissionRoutes must be initialized first');
    return root.path;
  }
  
  String get rootCompletePath {
    assert(_instance != null, 'CommissionRoutes must be initialized first');
    return root.completePath;
  }

  // Rotas do módulo
  static final BasePath root = BasePath(
    '/commission/',
    BasePath(_parentRoot),
  );
  
  static final BasePath dashboard = BasePath('/dashboard/', root);
  static final BasePath reports = BasePath('/reports/', root);
  static final BasePath calculations = BasePath('/calculations/', root);
  
  // Factory methods com validação
  static String buildReportPath(String reportType) {
    assert(_instance != null, 'CommissionRoutes must be initialized first');
    assert(['monthly', 'yearly', 'custom'].contains(reportType), 
           'Invalid report type: $reportType');
    return '${reports.completePath}?type=$reportType';
  }
}
```

---

## 📋 Templates para Routes

### Template - Rotas Internas

```dart
import 'package:base_core/base_core.dart';

import 'main_routes.dart'; // ou parent route apropriado

/// Rotas do módulo [ModuleName]
/// 
/// Define a navegação do módulo [descrição do módulo]
/// integrado à aplicação principal.
class [ModuleName]Routes {
  // Rota base do módulo
  static BasePath root = BasePath('/[module_path]/', MainRoutes.root);
  
  // Sub-rotas principais
  static BasePath [subRoute1] = BasePath('/[sub_path_1]/', root);
  static BasePath [subRoute2] = BasePath('/[sub_path_2]/', root);
  
  // Sub-rotas aninhadas (se necessário)
  static BasePath [nestedRoute] = BasePath('/[nested_path]/', [subRoute1]);
}
```

### Template - Rotas de Pacotes

```dart
import 'package:base_core/base_core.dart';

/// Rotas do pacote [PackageName]
/// 
/// [Descrição do pacote e sua funcionalidade]
/// Permite integração flexível através de configuração do parent root.
class [PackageName]Routes {
  // Private constructor para singleton
  [PackageName]Routes._internal({
    String parentRoot = '',
    // outros parâmetros de configuração
  });

  /// Inicializa e/ou obtém a instância do [PackageName]Routes
  /// 
  /// [parentRoot] define onde o módulo será integrado na hierarquia.
  /// [additionalParams] outros parâmetros de configuração.
  static [PackageName]Routes i({
    String parentRoot = '',
    // outros parâmetros
  }) {
    _parentRoot = parentRoot;
    // Configurar outros parâmetros
    
    _instance ??= [PackageName]Routes._internal(
      parentRoot: parentRoot,
      // outros parâmetros
    );
    return _instance!;
  }

  // Singleton instance
  static [PackageName]Routes? _instance;
  static late String? _parentRoot;

  // Getters de conveniência
  String get rootPath => root.path;
  String get rootCompletePath => root.completePath;

  // Rota base do pacote
  static final BasePath root = BasePath(
    '/[package_path]/',
    _parentRoot != null ? BasePath(_parentRoot!) : null,
  );
  
  // Sub-rotas do pacote
  static final BasePath [subRoute1] = BasePath('/[sub_path_1]/', root);
  static final BasePath [subRoute2] = BasePath('/[sub_path_2]/', root);
  
  // Métodos utilitários (opcionais)
  static String build[Entity]Path(String entityId) {
    return '${[subRoute1].completePath}?id=$entityId';
  }
}
```

### Convenções de Routes

**Nomenclatura:**
- Routes: `[ModuleName]Routes` (sempre sufixo Routes)
- Paths: camelCase para variáveis, snake_case para URLs
- Singletons: Método `.i()` para inicialização

**Estrutura:**
- Rotas internas: `static BasePath` simples
- Rotas de pacotes: Singleton com configuração
- Hierarquia clara: parent → child → grandchild
- Paths sempre começam e terminam com `/`

**Organização:**
- Uma classe por módulo/pacote
- Agrupamento lógico de rotas relacionadas
- Métodos utilitários para paths dinâmicos
- Validações quando necessário

---

## 📋 Checklist para Routes

### Checklist de Criação ✅

**Estrutura das Routes:**
- [ ] Escolha do tipo correto: Interna vs Pacote
- [ ] Nomenclatura seguindo padrão: `[Name]Routes`
- [ ] Localização apropriada: `/routes/` vs raiz do módulo
- [ ] Documentação clara da responsabilidade

**Rotas Internas:**
- [ ] Usa `static BasePath` simples
- [ ] Import do parent route correto
- [ ] Hierarquia clara e lógica
- [ ] Paths seguem convenção `/path/`

**Rotas de Pacotes:**
- [ ] Implementa singleton pattern corretamente
- [ ] Método `.i()` para inicialização
- [ ] Parent root configurável
- [ ] Getters de conveniência implementados
- [ ] Validações necessárias (se aplicável)

**Paths e Hierarquia:**
- [ ] Paths sempre começam e terminam com `/`
- [ ] Hierarquia lógica: root → sub-routes → nested
- [ ] Nomes descritivos e consistentes
- [ ] URLs amigáveis e RESTful

**Métodos Utilitários:**
- [ ] Factory methods para paths dinâmicos (se necessário)
- [ ] Validação de parâmetros (se aplicável)
- [ ] Assert statements para debuging
- [ ] Documentação dos métodos públicos

**Documentação:**
- [ ] Comentários explicando propósito do módulo
- [ ] Documentação dos parâmetros de configuração
- [ ] Exemplos de uso (para pacotes)
- [ ] Warnings sobre inicialização obrigatória

---

## 🎯 Diretrizes para Routes

### ✅ Boas Práticas

```dart
// ✅ Rotas internas simples e diretas
class ProfileRoutes {
  static BasePath root = BasePath('/profile/', HomeRoutes.root);
  static BasePath edit = BasePath('/edit/', root);
  static BasePath settings = BasePath('/settings/', root);
}

// ✅ Singleton bem implementado
class PaymentRoutes {
  PaymentRoutes._internal({String parentRoot = ''});
  
  static PaymentRoutes i({String parentRoot = ''}) {
    _parentRoot = parentRoot;
    _instance ??= PaymentRoutes._internal(parentRoot: parentRoot);
    return _instance!;
  }
  
  static PaymentRoutes? _instance;
  static late String? _parentRoot;
}

// ✅ Paths consistentes e limpos
static final BasePath checkout = BasePath('/checkout/', root);
static final BasePath success = BasePath('/success/', checkout);

// ✅ Factory methods úteis
static String buildOrderPath(String orderId) {
  return '${orderDetails.completePath}?id=$orderId';
}

// ✅ Validações apropriadas
static PaymentRoutes i({required String parentRoot}) {
  assert(parentRoot.isNotEmpty, 'Parent root cannot be empty');
  // ...
}

// ✅ Getters de conveniência
String get rootPath => root.path;
String get rootCompletePath => root.completePath;
```

### ❌ Evitar

```dart
// ❌ Misturar tipos (não usar singleton para rotas internas)
class InternalRoutes {
  static InternalRoutes? _instance; // ❌ Desnecessário para rotas internas
}

// ❌ Singleton mal implementado
class BadRoutes {
  static BadRoutes? instance; // ❌ Público
  
  static BadRoutes get i => instance ??= BadRoutes(); // ❌ Sem configuração
}

// ❌ Paths inconsistentes
static BasePath badPath1 = BasePath('no-slash-start', root); // ❌ Sem / inicial
static BasePath badPath2 = BasePath('/no-slash-end', root); // ❌ Sem / final

// ❌ Factory methods sem validação
static String buildPath(String id) {
  return '${root.completePath}?id=$id'; // ❌ Não valida se id não é vazio
}

// ❌ Não usar assert em configuração crítica
static PaymentRoutes i({String parentRoot = ''}) {
  // ❌ Aceita parent root vazio sem validar se é obrigatório
  _parentRoot = parentRoot;
  return _instance ??= PaymentRoutes._internal();
}

// ❌ Hierarquia confusa
static BasePath confusing = BasePath('/orders/', profileRoot); // ❌ Não faz sentido
```

---

## 🚀 Uso das Routes

### 1. Inicializando Rotas de Pacotes

```dart
// No main.dart ou setup da aplicação
void initializeRoutes() {
  // Inicializar rotas de pacotes com parent root
  FunnelRoutes.i(parentRoot: '/main/');
  CandidatesRoutes.i(parentRoot: '/main/', enableAnalytics: true);
  CommissionRoutes.i(parentRoot: '/main/', apiVersion: 'v2');
}
```

### 2. Navegação com Routes

```dart
// Usando rotas internas
void navigateToProfile() {
  Nav.to.push(ProfileRoutes.root.completePath);
}

// Usando rotas de pacotes
void navigateToFunnel() {
  Nav.to.push(FunnelRoutes.i().root.completePath);
}

// Usando factory methods
void navigateToEnrollmentDetails(String enrollmentId) {
  final path = CandidatesRoutes.buildEnrollmentDetailsPath(enrollmentId);
  Nav.to.push(path);
}
```

### 3. Configuração Condicional

```dart
// Configuração baseada em ambiente
void setupEnvironmentRoutes() {
  final parentRoot = kDebugMode ? '/debug/' : '/main/';
  
  FunnelRoutes.i(parentRoot: parentRoot);
  CandidatesRoutes.i(
    parentRoot: parentRoot,
    enableAnalytics: !kDebugMode,
  );
}
```

### 4. Integração com Router

```dart
// Configuração do Go Router
final router = GoRouter(
  routes: [
    GoRoute(
      path: HomeRoutes.root.path,
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: ProfileRoutes.edit.path,
          builder: (context, state) => const EditProfilePage(),
        ),
      ],
    ),
    GoRoute(
      path: FunnelRoutes.i().root.path,
      builder: (context, state) => const FunnelHomePage(),
    ),
  ],
);
```

---

## 🎯 Padrões Avançados

### 1. Routes com Middleware

```dart
class SecureRoutes {
  SecureRoutes._internal({
    String parentRoot = '',
    this.requireAuth = true,
  });

  static SecureRoutes i({
    String parentRoot = '',
    bool requireAuth = true,
  }) {
    _parentRoot = parentRoot;
    _requireAuth = requireAuth;
    
    _instance ??= SecureRoutes._internal(
      parentRoot: parentRoot,
      requireAuth: requireAuth,
    );
    return _instance!;
  }

  static SecureRoutes? _instance;
  static late String _parentRoot;
  static late bool _requireAuth;

  final bool requireAuth;

  static final BasePath root = BasePath('/secure/', BasePath(_parentRoot!));
  static final BasePath admin = BasePath('/admin/', root);
  
  // Validação de acesso
  static bool canAccessRoute(String routePath) {
    if (!_requireAuth) return true;
    
    // Lógica de validação de acesso
    return AuthService.isAuthenticated && 
           AuthService.hasPermissionFor(routePath);
  }
}
```

### 2. Routes com Configuração Dinâmica

```dart
class DynamicRoutes {
  DynamicRoutes._internal({
    required Map<String, String> customPaths,
  }) : _customPaths = customPaths;

  static DynamicRoutes i({
    String parentRoot = '',
    Map<String, String> customPaths = const {},
  }) {
    _parentRoot = parentRoot;
    
    _instance ??= DynamicRoutes._internal(
      customPaths: customPaths,
    );
    return _instance!;
  }

  static DynamicRoutes? _instance;
  static late String _parentRoot;
  
  final Map<String, String> _customPaths;

  static late final BasePath root = BasePath(
    _instance!._customPaths['root'] ?? '/dynamic/',
    BasePath(_parentRoot),
  );
  
  String getCustomPath(String key) {
    return _customPaths[key] ?? '/default/';
  }
}
```

### 3. Routes com Versionamento

```dart
class VersionedRoutes {
  VersionedRoutes._internal({
    required this.version,
    String parentRoot = '',
  });

  static VersionedRoutes i({
    required String version,
    String parentRoot = '',
  }) {
    final key = '$parentRoot-$version';
    _instances[key] ??= VersionedRoutes._internal(
      version: version,
      parentRoot: parentRoot,
    );
    return _instances[key]!;
  }

  static final Map<String, VersionedRoutes> _instances = {};
  
  final String version;

  BasePath get root => BasePath('/api/$version/', BasePath(_parentRoot));
  BasePath get users => BasePath('/users/', root);
  BasePath get products => BasePath('/products/', root);
}
```

---

## 🎯 Resumo dos Benefícios

### ✅ **Organização Clara**
- **Dois padrões distintos**: Internas simples, pacotes configuráveis
- **Hierarquia bem definida**: Parent → child → grandchild
- **Separação por contexto**: Cada módulo tem suas próprias rotas
- **Paths consistentes**: Convenções claras de nomenclatura

### ✅ **Flexibilidade de Integração**
- **Rotas de pacotes configuráveis**: Parent root dinâmico
- **Singleton controlado**: Inicialização única e consistente
- **Validações apropriadas**: Assert statements para configuração
- **Factory methods**: Paths dinâmicos e type-safe

### ✅ **Developer Experience**
- **Templates reutilizáveis**: Estrutura padrão para acelerar desenvolvimento
- **Type safety**: BasePath tipificado previne erros
- **Getters convenientes**: rootPath, rootCompletePath
- **Documentação clara**: Propósito e uso bem documentados

### ✅ **Maintainability**
- **Padrões consistentes**: Mesmo approach em toda aplicação
- **Facilita refactoring**: Mudanças centralizadas nas routes
- **Testável**: Paths bem definidos facilitam testes
- **Escalável**: Fácil adição de novos módulos e rotas

Esta arquitetura de Routes garante **navegação organizada**, **integração flexível** e **manutenibilidade** em aplicações Flutter modulares! 🎯
