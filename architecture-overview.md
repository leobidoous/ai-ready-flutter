# Architecture Overview - Cogna Resale Core

## 📋 Visão Geral

O **cogna-resale-core** é um pacote Flutter que implementa a **Clean Architecture** com separação rigorosa de responsabilidades em camadas. Este documento descreve a arquitetura completa, princípios SOLID aplicados, fluxo de dependências e padrões utilizados.

---

## 🏗️ Estrutura de Camadas

```
lib/src/
├── domain/          # Camada de Domínio (Contratos e Regras de Negócio)
├── infra/           # Camada de Infraestrutura (Implementações e Coordenação)
├── data/            # Camada de Dados (Comunicação Externa)
├── presentation/    # Camada de Apresentação (UI e Estado)
└── core/            # Utilitários Compartilhados
```

### Fluxo de Dependências

```
Presentation → Infrastructure → Data
     ↓              ↓
   Domain ← ← ← ← ← ←
```

**Regra de Ouro:** Todas as camadas dependem do **Domain**, nunca de implementações concretas.

---

## 📦 Camada de Domínio (Domain)

### Responsabilidades

- Define **O QUE** a aplicação faz (contratos puros)
- Contém **regras de negócio** fundamentais
- **Zero dependências** externas (exceto base_core)
- Totalmente **testável** e **reutilizável**

### Estrutura

```
domain/
├── entities/        # Objetos de negócio puros
├── enums/           # Valores constantes tipados
├── failures/        # Tipos de erro do domínio
├── repositories/    # Contratos de acesso a dados
├── usecases/        # Contratos de operações de negócio
└── services/        # Contratos de serviços auxiliares
```

### Entities (Objetos de Negócio)

**Características:**

- Imutáveis (`const` constructors)
- Campos `final`
- Método `copyWith` para modificações
- Sem lógica de serialização (fica nos Models)
- Composição de outras entities

**Exemplo:**

```dart
class UserEntity {
  const UserEntity({
    required this.id,
    required this.name,
    required this.email,
    required this.cpf,
    required this.birth,
    required this.status,
    required this.gender,
    required this.role,
    required this.address,
    required this.company,
    required this.partnerCompany,
    required this.notificationPreferences,
  });

  final String id;
  final String name;
  final String email;
  final String cpf;
  final DateTime birth;
  final UserStatus status;
  final UserGenderType gender;
  final UserRoleEntity role;
  final AddressEntity address;
  final UserCompanyEntity company;
  final DistributorEntity partnerCompany;
  final UserNotificationPreferencesEntity notificationPreferences;

  UserEntity copyWith({
    String? id,
    String? name,
    // ... outros campos
  }) {
    return UserEntity(
      id: id ?? this.id,
      name: name ?? this.name,
      // ... outros campos
    );
  }
}
```

### Enums (Valores Constantes)

**Características:**

- Enhanced Enums do Dart
- Campos `final` com metadados
- Métodos `toJson` e `fromJson`
- Exhaustiveness checking (switch expressions)

**Exemplo:**

```dart
enum SystemType {
  dc(name: 'Distribuidor Cogna', toJson: 'DC'),
  ce(name: 'Consultoria Educação', toJson: 'CE'),
  captarExpress(name: 'Captar Express', toJson: 'CAPTAR_EXPRESS');

  const SystemType({required this.name, required this.toJson});

  final String name;
  final String toJson;

  static SystemType fromJson(String? type) {
    return values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => SystemType.ce,
    );
  }
}
```

### Failures (Tipos de Erro)

**Características:**

- Herdam de `ICustomFailure` (base_core)
- Interface abstrata por contexto
- Classes concretas para erros específicos
- Usados com `Either<Failure, Success>`

**Exemplo:**

```dart
abstract class IUserFailure extends ICustomFailure {
  IUserFailure({required super.message});
}

class UserServerError extends IUserFailure {
  UserServerError({required super.message});
}

class UserUnknownError extends IUserFailure {
  UserUnknownError({required super.message});
}

class UserUnauthenticatedError extends IUserFailure {
  UserUnauthenticatedError({required super.message});
}
```

### Repository Interfaces (Contratos de Acesso a Dados)

**Características:**

- Prefixo `I` (ex: `IUserRepository`)
- Retornam `Either<Failure, Success>`
- Trabalham com Entities
- Abstraem origem dos dados

**Exemplo:**

```dart
abstract class IUserRepository {
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
    required UserFiltersEntity filters,
  });
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  });
  Future<Either<IUserFailure, Unit>> deleteUserById({required String id});
}
```

### UseCase Interfaces (Contratos de Operações de Negócio)

**Características:**

- Prefixo `I` (ex: `IUserUsecase`)
- Representam casos de uso específicos
- Orquestram Repositories
- Aplicam validações de negócio

**Exemplo:**

```dart
abstract class IUserUsecase {
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });
}
```

---

## 🔧 Camada de Infraestrutura (Infra)

### Responsabilidades

- Implementa **COMO** as operações são executadas
- Coordena transformação de dados
- Conecta Domain com Data
- Não conhece detalhes de comunicação externa

### Estrutura

```
infra/
├── datasources/     # Interfaces de comunicação externa
├── drivers/         # Interfaces de bibliotecas externas
├── models/          # Serialização/Deserialização
├── repositories/    # Implementações de repositórios
├── usecases/        # Implementações de casos de uso
└── services/        # Implementações de serviços
```

### DataSource Interfaces

**Características:**

- Prefixo `I` (ex: `IUserDatasource`)
- Retornam `Either<HttpErrorResponse, HttpDriverResponse>`
- Trabalham com dados brutos (JSON)
- Abstraem protocolo de comunicação

**Exemplo:**

```dart
abstract class IUserDatasource {
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser();
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUserById({
    required String id,
  });
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  });
}
```

### Models (Serialização)

**Características:**

- Estendem Entities
- Mixin `EquatableMixin`
- Métodos `fromMap`, `toMap`, `toCreate`, `toUpdate`
- Tratam dados nulos e formatação

**Exemplo:**

```dart
class UserModel extends UserEntity with EquatableMixin {
  UserModel({
    required super.id,
    required super.name,
    required super.email,
    // ... outros campos
  });

  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      id: map['id']?.toString() ?? '',
      name: map['name']?.toString() ?? '',
      email: map['email']?.toString() ?? '',
      cpf: map['cpf']?.toString() ?? '',
      birth: DateFormat.tryParseOrDateNow(map['birthDate']?.toString()),
      status: UserStatus.fromJson(map['status']?.toString()),
      gender: UserGenderType.fromJson(map['gender']?.toString()),
      role: UserRoleModel.fromMap(
        map['role'] is Map ? map['role'] : {},
      ),
      address: AddressModel.fromMap(
        map['address'] is Map ? map['address'] : {},
      ),
      // ... outros campos
    );
  }

  factory UserModel.fromEntity(UserEntity entity) {
    return UserModel(
      id: entity.id,
      name: entity.name,
      // ... outros campos
    );
  }

  UserEntity get toEntity => this;

  Map<String, dynamic> get toMap {
    return {
      'id': id,
      'name': name,
      'email': email,
      'cpf': cpf.replaceAll(RegExp('[^0-9]'), ''),
      'birthDate': birth.toIso8601String(),
      'status': status.toJson,
      'gender': gender.toJson,
      // ... outros campos
    };
  }

  Map<String, dynamic> get toCreate {
    return {
      'name': name,
      'email': email,
      'cpf': cpf.trim().replaceAll(RegExp('[^0-9]'), ''),
      // ... campos necessários para criação
    };
  }

  @override
  List<Object?> get props => [id, name, email, cpf, birth, status, gender];

  @override
  bool? get stringify => true;
}
```

### Repository Implementations

**Características:**

- Implementam interfaces do Domain
- Coordenam DataSources
- Transformam dados (Model ↔ Entity)
- Transformam erros (HTTP → Failure)

**Exemplo:**

```dart
class UserRepository extends IUserRepository {
  UserRepository({
    required this.crashLog,
    required this.datasource,
    required this.firebaseAuthService,
  });

  final CrashLog crashLog;
  final IUserDatasource datasource;
  final IFirebaseAuthService firebaseAuthService;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    return datasource.getLoggedUser().then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) {
    return datasource.updateUser(data: data).then((value) {
      try {
        return value.fold(
          (l) => Left(UserServerError(message: l.message)),
          (r) => Right(UserModel.fromMap(r.data)),
        );
      } catch (exception, stackTrace) {
        crashLog.capture(exception: exception, stackTrace: stackTrace);
        return Left(UserUnknownError(message: '$exception'));
      }
    });
  }
}
```

### UseCase Implementations

**Características:**

- Implementam interfaces do Domain
- Delegam para Repositories
- Aplicam validações de negócio
- Orquestram múltiplos repositories quando necessário

**Exemplo:**

```dart
class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});

  final IUserRepository repository;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    return repository.getLoggedUser();
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) {
    // Pode adicionar validações aqui antes de chamar o repository
    return repository.updateUser(data: data);
  }
}
```

---

## 📡 Camada de Dados (Data)

### Responsabilidades

- Executa **COMUNICAÇÃO** real com APIs, DB, cache
- Implementa DataSources
- Usa Drivers para abstrair bibliotecas externas
- Não conhece regras de negócio

### Estrutura

```
data/
├── datasources/     # Implementações de comunicação
├── drivers/         # Implementações de drivers
└── interceptors/    # Interceptors HTTP (auth, logging, etc.)
```

### DataSource Implementations

**Características:**

- Implementam interfaces da Infra
- Usam `IHttpDriver` para comunicação
- Transformam Entities em dados para envio
- Retornam dados brutos (JSON)

**Exemplo:**

```dart
class UserDatasource extends IUserDatasource {
  UserDatasource({required this.httpDriver});

  final IHttpDriver httpDriver;

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser() async {
    return httpDriver
        .get('/api/v1/user/me')
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  }) async {
    final formData = FormData.fromMap(
      await UserModel.fromEntity(data).toUpdateAsync,
    );
    return httpDriver
        .patch('/api/v1/user/me', data: formData)
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
    required UserEntity data,
  }) {
    return httpDriver
        .post('/api/v1/user', data: UserModel.fromEntity(data).toCreate)
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

### Drivers

**Características:**

- Abstraem bibliotecas externas (Dio, SharedPreferences, etc.)
- Fornecem interface padronizada
- Isolam dependências externas

**Exemplo:**

```dart
abstract class IHttpDriver {
  Future<Either<HttpDriverResponse, HttpDriverResponse>> get(
    String path, {
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> post(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> patch(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });

  Future<Either<HttpDriverResponse, HttpDriverResponse>> delete(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    HttpDriverOptions? options,
  });
}
```

---

## 🎨 Camada de Apresentação (Presentation)

### Responsabilidades

- Gerencia **ESTADO DA UI**
- Coordena chamadas aos UseCases
- Não contém lógica de negócio
- Reage a mudanças de estado

### Estrutura

```
presentation/
├── controllers/     # Gerenciamento de estado
├── modules/         # Injeção de dependências (Flutter Modular)
├── pages/           # Telas da aplicação
├── widgets/         # Componentes reutilizáveis
├── extensions/      # Extensions do Flutter
├── mixins/          # Mixins reutilizáveis
├── router_guards/   # Guards de navegação
└── services/        # Serviços de UI (tutorial, etc.)
```

### Controllers

**Características:**

- Estendem `CustomController<Failure, Entity>`
- Gerenciam estado reativo (ValueNotifier)
- Método `execute()` para operações assíncronas
- Estados: initial, loading, success, error

**Exemplo:**

```dart
class UserController extends CustomController<IUserFailure, UserEntity> {
  UserController({required this.usecase}) : super(UserModel.fromMap({}));

  final IUserUsecase usecase;

  UserEntity data = UserModel.fromMap({});

  Future<void> getLoggedUser() async {
    await execute(() => usecase.getLoggedUser());
  }

  Future<void> updateUser(UserEntity data) async {
    await execute(() => usecase.updateUser(data: data));
  }
}
```

**Uso na UI:**

```dart
class UserPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = context.watch<UserController>();

    return controller.state.when(
      initial: () => Text('Carregando...'),
      loading: () => CircularProgressIndicator(),
      success: (user) => Text('Olá, ${user.name}'),
      error: (failure) => Text('Erro: ${failure.message}'),
    );
  }
}
```

### Modules (Injeção de Dependências)

**Características:**

- Estendem `Module` (Flutter Modular)
- Registram todas as dependências
- Configuram rotas e guards
- Importam outros módulos

**Exemplo:**

```dart
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
  }
}
```

---

## 🛠️ Camada Core (Utilitários)

### Responsabilidades

- Utilitários compartilhados
- Constantes da aplicação
- Validadores e máscaras
- Formatadores
- Temas

### Estrutura

```
core/
├── constants/       # Constantes da aplicação
├── formats/         # Formatadores (data, moeda, etc.)
├── masks/           # Máscaras de input (CPF, telefone, etc.)
├── themes/          # Temas da aplicação
├── utils/           # Utilitários gerais
└── validators/      # Validadores de formulário
```

---

## 🎯 Princípios SOLID Aplicados

### Single Responsibility Principle (SRP)

- Cada classe tem uma única responsabilidade
- Controllers apenas gerenciam estado
- UseCases apenas orquestram lógica de negócio
- Repositories apenas coordenam acesso a dados
- DataSources apenas comunicam com APIs

### Open/Closed Principle (OCP)

- Aberto para extensão, fechado para modificação
- Novas implementações sem alterar interfaces
- Novos DataSources sem alterar Repositories

### Liskov Substitution Principle (LSP)

- Implementações podem substituir interfaces
- `UserRepository` substitui `IUserRepository`
- `UserDatasource` substitui `IUserDatasource`

### Interface Segregation Principle (ISP)

- Interfaces específicas e focadas
- `IUserRepository` não tem métodos de `IDistributorRepository`
- Clients não dependem de métodos que não usam

### Dependency Inversion Principle (DIP)

- Camadas dependem de abstrações (interfaces)
- Domain não conhece implementações
- Presentation depende de `IUserUsecase`, não de `UserUsecase`

---

## 📊 Fluxo Completo de Dados

### Fluxo de Leitura (GET)

```
1. UI (Widget)
   ↓ chama método
2. Controller
   ↓ execute(() => usecase.method())
3. UseCase
   ↓ repository.method()
4. Repository
   ↓ datasource.method()
5. DataSource
   ↓ httpDriver.get()
6. Driver (Dio)
   ↓ HTTP GET
7. API
   ↓ JSON response
8. Driver
   ↓ HttpDriverResponse
9. DataSource
   ↓ Either<HttpError, HttpResponse>
10. Repository
    ↓ transforma em Either<Failure, Entity>
11. UseCase
    ↓ Either<Failure, Entity>
12. Controller
    ↓ atualiza state
13. UI
    ↓ reage ao state (success/error)
```

### Fluxo de Escrita (POST/PATCH)

```
1. UI (Widget)
   ↓ chama método com Entity
2. Controller
   ↓ execute(() => usecase.method(entity))
3. UseCase
   ↓ repository.method(entity)
4. Repository
   ↓ datasource.method(entity)
5. DataSource
   ↓ Model.fromEntity(entity).toCreate
   ↓ httpDriver.post(data: map)
6. Driver (Dio)
   ↓ HTTP POST com JSON
7. API
   ↓ JSON response
8. Driver
   ↓ HttpDriverResponse
9. DataSource
   ↓ Either<HttpError, HttpResponse>
10. Repository
    ↓ Model.fromMap(response.data)
    ↓ Either<Failure, Entity>
11. UseCase
    ↓ Either<Failure, Entity>
12. Controller
    ↓ atualiza state
13. UI
    ↓ reage ao state (success/error)
```

---

## 🧪 Testabilidade

### Vantagens da Arquitetura

1. **Isolamento de Camadas**
   - Cada camada pode ser testada independentemente
   - Mocks fáceis de criar (interfaces)

2. **Injeção de Dependências**
   - Substituir implementações por mocks
   - Testar comportamento sem dependências externas

3. **Either Pattern**
   - Tratamento explícito de erros
   - Testes de casos de sucesso e falha

### Exemplo de Teste

```dart
// Mock do Repository
class MockUserRepository implements IUserRepository {
  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
    return Right(UserEntity(/* dados de teste */));
  }
}

// Teste do UseCase
test('should return user when repository succeeds', () async {
  // Arrange
  final repository = MockUserRepository();
  final usecase = UserUsecase(repository: repository);

  // Act
  final result = await usecase.getLoggedUser();

  // Assert
  expect(result.isRight(), true);
  expect(result.getRight().toNullable()?.name, 'Test User');
});
```

---

## 📝 Convenções de Nomenclatura

### Interfaces

- Prefixo `I` para todas as interfaces
- Exemplos: `IUserRepository`, `IUserUsecase`, `IUserFailure`

### Implementações

- Sem prefixo, nome descritivo
- Exemplos: `UserRepository`, `UserUsecase`, `UserServerError`

### Models

- Sufixo `Model`
- Exemplo: `UserModel`, `UserResultModel`

### Entities

- Sufixo `Entity`
- Exemplo: `UserEntity`, `AddressEntity`

### Controllers

- Sufixo `Controller`
- Exemplo: `UserController`, `UsersController`

### Enums

- Sufixo `Type` ou `Status`
- Exemplo: `SystemType`, `UserStatus`, `UserGenderType`

---

## 🔗 Padrões Utilizados

### Either Pattern

- Representa operações que podem falhar
- `Left`: Erro (Failure)
- `Right`: Sucesso (Data)
- Força tratamento explícito de erros

### Repository Pattern

- Abstrai acesso a dados
- Isola lógica de negócio de detalhes de implementação

### UseCase Pattern

- Encapsula operações de negócio
- Um UseCase = Uma operação específica

### Dependency Injection

- Injeção via construtor
- Facilita testes e desacoplamento
- Flutter Modular para DI

### ValueNotifier Pattern

- Estado reativo na UI
- CustomController gerencia estados
- UI reage automaticamente a mudanças

---

## 🚀 Benefícios da Arquitetura

### Manutenibilidade

- Código organizado e previsível
- Mudanças isoladas em camadas específicas
- Fácil localizar e corrigir bugs

### Escalabilidade

- Adicionar novos módulos sem afetar existentes
- Reutilização de código entre projetos
- Crescimento sustentável do projeto

### Testabilidade

- Cada camada testável independentemente
- Mocks fáceis de criar
- Cobertura de testes alta

### Flexibilidade

- Trocar implementações sem afetar outras camadas
- Migrar de Dio para http sem afetar Domain
- Adicionar novos DataSources facilmente

### Colaboração

- Estrutura clara para novos desenvolvedores
- Padrões consistentes em todo o projeto
- Documentação viva (código auto-explicativo)

---

## 📚 Próximos Passos

Para entender melhor cada camada, consulte a documentação específica:

1. **[Domain Layer](./domain/)** - Entities, Enums, Failures, Repositories, UseCases
2. **[Infrastructure Layer](./infra/)** - Models, Repositories, UseCases, DataSources
3. **[Data Layer](./data/)** - DataSources, Drivers
4. **[Presentation Layer](./presentation/)** - Controllers, Modules, Pages, Widgets

---

_Esta arquitetura garante código limpo, testável e escalável, seguindo os princípios SOLID e as melhores práticas da Clean Architecture._
