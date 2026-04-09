# Modules (Presentation)

## O que são?

Modules são responsáveis por **organizar e injetar dependências** de um módulo funcional da aplicação. Eles agrupam DataSources, Repositories, UseCases, Controllers e Rotas relacionadas.

## Responsabilidades

- Registrar todas as dependências do módulo
- Configurar rotas e navegação
- Importar outros módulos necessários
- Definir guards de autenticação/autorização
- Organizar a estrutura modular da aplicação

## Estrutura

```dart
import 'package:cogna_resale_core/cogna_resale_core.dart'
    show
        HttpModule,
        Injector,
        Module,
        RouteManager,
        TransitionType,
        UserRoleType,
        UserRoleTypeRouterGuard;

import '../../data/datasources/user_datasource.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/usecases/i_user_usecase.dart';
import '../../infra/datasources/i_user_datasource.dart';
import '../../infra/repositories/user_repository.dart';
import '../../infra/usecases/user_usecase.dart';
import '../controllers/user_controller.dart';
import '../controllers/users_controller.dart';
import '../pages/user/user_profile_page.dart';
import '../pages/user/user_edit_page.dart';
import '../pages/user/users_list_page.dart';
import '../routes/user_routes.dart';

class UserModule extends Module {
  @override
  List<Module> get imports => [HttpModule()];

  @override
  void binds(Injector i) {
    // DataSources
    i.add<IUserDatasource>(UserDatasource.new);

    // Repositories
    i.add<IUserRepository>(UserRepository.new);

    // UseCases
    i.add<IUserUsecase>(UserUsecase.new);

    // Controllers
    i.add<UserController>(UserController.new);
    i.add<UsersController>(UsersController.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      UserRoutes.profile.path,
      transition: .fadeIn,
      child: (_) => const UserProfilePage(),
    );

    r.child(
      UserRoutes.edit.path,
      transition: .fadeIn,
      child: (_) => const UserEditPage(),
      guards: [
        UserRoleTypeRouterGuard(
          roles: <UserRoleType>[.admin, .director],
          redirectTo: UserRoutes.profile.completePath,
        ),
      ],
    );

    r.child(
      UserRoutes.list.path,
      transition: .fadeIn,
      child: (_) => const UsersListPage(),
      guards: [
        UserRoleTypeRouterGuard(
          roles: <UserRoleType>[.admin],
          redirectTo: UserRoutes.profile.completePath,
        ),
      ],
    );
  }
}
```

## Características

### 1. Herança de Module

```dart
class UserModule extends Module
```

- Estende classe base `Module`
- Fornece estrutura para injeção e rotas

### 2. Imports - Módulos Dependentes

```dart
@override
List<Module> get imports => [HttpModule()];
```

- Lista módulos que este módulo depende
- Dependências são carregadas automaticamente
- Exemplo: `HttpModule` fornece `IHttpDriver`

### 3. Binds - Injeção de Dependências

```dart
@override
void binds(Injector i) {
  // DataSources
  i.add<IUserDatasource>(UserDatasource.new);

  // Repositories
  i.add<IUserRepository>(UserRepository.new);

  // UseCases
  i.add<IUserUsecase>(UserUsecase.new);

  // Controllers
  i.add<UserController>(UserController.new);
  i.add<UsersController>(UsersController.new);
}
```

#### Ordem de Registro:

1. **DataSources** (camada mais baixa)
2. **Repositories** (dependem de DataSources)
3. **UseCases** (dependem de Repositories)
4. **Controllers** (dependem de UseCases)

#### Sintaxe de Registro:

```dart
i.add<Interface>(Implementation.new);
```

- Primeiro tipo genérico: Interface ou classe abstrata
- Segundo parâmetro: Constructor reference da implementação
- Injetor resolve dependências automaticamente

### 4. Routes - Configuração de Rotas

#### Rota Simples

```dart
r.child(
  UserRoutes.profile.path,
  transition: .fadeIn,
  child: (_) => const UserProfilePage(),
);
```

#### Rota com Guards

```dart
r.child(
  UserRoutes.edit.path,
  transition: .fadeIn,
  child: (_) => const UserEditPage(),
  guards: [
    UserRoleTypeRouterGuard(
      roles: <UserRoleType>[.admin, .director],
      redirectTo: UserRoutes.profile.completePath,
    ),
  ],
);
```

#### Rota com Permissão Restrita

```dart
r.child(
  UserRoutes.list.path,
  transition: .fadeIn,
  child: (_) => const UsersListPage(),
  guards: [
    UserRoleTypeRouterGuard(
      roles: <UserRoleType>[.admin],
      redirectTo: UserRoutes.profile.completePath,
    ),
  ],
);
```

## Padrão de Injeção

### Resolução Automática

O Injector resolve dependências automaticamente seguindo a ordem:

```dart
// 1. DataSource precisa de HttpDriver (vem do HttpModule)
i.add<IUserDatasource>(UserDatasource.new);

// 2. Repository precisa de DataSource (registrado acima)
i.add<IUserRepository>(UserRepository.new);

// 3. UseCase precisa de Repository (registrado acima)
i.add<IUserUsecase>(UserUsecase.new);

// 4. Controller precisa de UseCase (registrado acima)
i.add<UserController>(UserController.new);
```

### Exemplo de Resolução:

```dart
// Quando UserController é criado:
UserController(
  usecase: UserUsecase(
    repository: UserRepository(
      crashLog: CrashLog(),
      datasource: UserDatasource(
        httpDriver: HttpDriver(), // Do HttpModule
      ),
      firebaseAuthService: FirebaseAuthService(),
    ),
  ),
)
```

## Guards de Rota

### UserRoleTypeRouterGuard

```dart
UserRoleTypeRouterGuard(
  roles: <UserRoleType>[.admin, .director],
  redirectTo: UserRoutes.profile.completePath,
)
```

- Verifica se usuário tem permissão
- `roles`: Lista de roles permitidas
- `redirectTo`: Rota de redirecionamento se não autorizado

### Exemplo de Permissões:

```dart
// Apenas administradores gerais
roles: <UserRoleType>[.admin]

// Administradores e comerciais
roles: <UserRoleType>[.admin, .director]

// Todos os tipos de usuário
roles: <UserRoleType>[.admin, .director, .consultantPj, .consultantPf]
```

## Transições de Rota

```dart
.fadeIn      // Fade suave
.slideLeft   // Desliza da direita
.slideRight  // Desliza da esquerda
.slideUp     // Desliza de baixo
.slideDown   // Desliza de cima
```

## Organização de Imports

### Ordem Recomendada:

```dart
// 1. Packages externos
import 'package:cogna_resale_core/cogna_resale_core.dart';

// 2. Data layer
import '../../data/datasources/user_datasource.dart';

// 3. Domain layer
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/usecases/i_user_usecase.dart';

// 4. Infra layer
import '../../infra/datasources/i_user_datasource.dart';
import '../../infra/repositories/user_repository.dart';
import '../../infra/usecases/user_usecase.dart';

// 5. Presentation layer
import '../controllers/user_controller.dart';
import '../pages/user/user_profile_page.dart';
import '../routes/user_routes.dart';
```

## Fluxo Completo de Injeção

### 1. Registro no Module

```dart
class UserModule extends Module {
  @override
  void binds(Injector i) {
    i.add<IUserDatasource>(UserDatasource.new);
    i.add<IUserRepository>(UserRepository.new);
    i.add<IUserUsecase>(UserUsecase.new);
    i.add<UserController>(UserController.new);
  }
}
```

### 2. Construtor do UserDatasource

```dart
class UserDatasource extends IUserDatasource {
  UserDatasource({required this.httpDriver});

  final IHttpDriver httpDriver; // Injetado do HttpModule
}
```

### 3. Construtor do UserRepository

```dart
class UserRepository extends IUserRepository {
  UserRepository({
    required this.crashLog,
    required this.datasource,
    required this.firebaseAuthService,
  });

  final CrashLog crashLog;
  final IUserDatasource datasource; // Injetado do registro acima
  final IFirebaseAuthService firebaseAuthService;
}
```

### 4. Construtor do UserUsecase

```dart
class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});

  final IUserRepository repository; // Injetado do registro acima
}
```

### 5. Construtor do UserController

```dart
class UserController extends CustomController<IUserFailure, UserEntity> {
  UserController({required this.usecase}) : super(UserModel.fromMap({}));

  final IUserUsecase usecase; // Injetado do registro acima
}
```

### 6. Uso na UI

```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Injector resolve automaticamente toda a cadeia de dependências
    final controller = context.watch<UserController>();

    return Scaffold(
      body: controller.state.when(
        success: (user) => Text('Olá, ${user.name}'),
        error: (failure) => Text('Erro: ${failure.message}'),
        loading: () => CircularProgressIndicator(),
      ),
    );
  }
}
```

## Benefícios

- **Organização**: Agrupa código relacionado
- **Modularidade**: Módulos independentes e reutilizáveis
- **Injeção automática**: Resolve dependências automaticamente
- **Lazy loading**: Módulos carregados sob demanda
- **Testabilidade**: Fácil substituir dependências em testes
- **Manutenibilidade**: Mudanças isoladas em módulos específicos
- **Escalabilidade**: Adicionar novos módulos sem afetar existentes
- **Segurança**: Guards controlam acesso às rotas
