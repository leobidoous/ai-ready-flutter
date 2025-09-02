# Infrastructure DataSources (Interfaces) - Clean Architecture

## 📚 Visão Geral

Esta documentação define as **interfaces de datasources** da camada de **Infrastructure** em nossa arquitetura limpa. As interfaces estabelecem **COMO** deve ser feita a comunicação com fontes de dados externas, definindo contratos de protocolos sem implementações específicas.

### 🎯 Princípios Fundamentais das Interfaces DataSource

**O QUE as interfaces DEFINEM:**
- ✅ **Contratos de Comunicação**: COMO comunicar com fontes externas (protocolo)
- ✅ **Assinaturas de Protocolos**: Métodos sem implementação, apenas contratos
- ✅ **Especificação de I/O**: Parâmetros de entrada e saída tipados fortemente  
- ✅ **Tratamento de Erros**: Either pattern para falhas de comunicação
- ✅ **Documentação de Protocolos**: Endpoints, headers, formatos de dados

**O QUE as interfaces NÃO FAZEM:**
- ❌ **Não implementam comunicação real**: Apenas definem contratos
- ❌ **Não dependem de tecnologias específicas**: Não especificam Dio, Retrofit, etc.
- ❌ **Não contêm lógica de parsing**: Zero detalhes de serialização
- ❌ **Não implementam autenticação**: Apenas documentam necessidade
- ❌ **Não quebram SOLID**: Dependem de abstrações, não implementações

```

---

## 🔒 Princípios SOLID em Interfaces DataSource

### 1. **Dependency Inversion Principle (DIP)**
```dart
// ✅ Interface depende apenas de abstrações
import '../../domain/entities/user_entity.dart';     // Abstração do domain
import '../../domain/failures/i_user_failures.dart'; // Abstração do domain

abstract class IUserDatasource {
  // Recebe entities (abstrações), retorna failures (abstrações)
  Future<Either<IUserFailure, UserEntity>> getUser();
}

// ❌ NUNCA depender de implementações concretas
// import 'package:dio/dio.dart';            // Implementação específica de HTTP
// import 'package:sqflite/sqflite.dart';    // Implementação específica de DB
```

### 2. **Interface Segregation Principle (ISP)**
```dart
// ✅ Interfaces específicas por tipo de comunicação
abstract class IUserRemoteDatasource {
  Future<Either<IUserFailure, UserEntity>> getUserFromApi();
}

abstract class IUserLocalDatasource {
  Future<Either<IUserFailure, UserEntity>> getUserFromCache();
}

abstract class IUserDatabaseDatasource {
  Future<Either<IUserFailure, UserEntity>> getUserFromDB();
}

// ❌ Interface única misturando protocolos diferentes
abstract class IUserEverythingDatasource {
  // API + DB + Cache + File... EVITAR!
}
```

### 3. **Tipagem Forte e Either Pattern Obrigatório**
```dart
// ✅ Tipagem forte com Either para comunicação externa
abstract class IUserDatasource {
  /// SEMPRE Either<Failure, Success> pois comunicação externa pode falhar
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id, // Tipagem forte e obrigatória
  });
  
  /// Documentar protocolo de comunicação
  Future<Either<IUserFailure, Unit>> updateUser({
    required UserEntity data,
    Map<String, String>? headers, // Parâmetros opcionais para protocolo
  });
}

// ❌ Evitar comunicação sem tratamento de erro
// Future<UserEntity> getUser();           // Comunicação externa pode falhar
// UserEntity getUserSync();               // I/O externo deve ser assíncrono
// Future<Map<String, dynamic>> getUser(); // Retorno não tipado
```

---

## 🏗️ Estrutura de Arquivos

---

## 🔍 Anatomia de uma Interface DataSource

### Componentes Principais

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../domain/entities/user_entity.dart';
import '../../domain/failures/i_user_failures.dart';

/// Interface que define o contrato para comunicação com fontes de dados de usuários
/// 
/// Esta interface estabelece as operações de comunicação para:
/// - Acesso a APIs REST para dados de usuários
/// - Comunicação com banco de dados
/// - Operações de cache quando aplicável
/// - Sincronização com sistemas externos
abstract class IUserDatasource {
  // Métodos definindo contratos de comunicação com fontes externas
}
```

### Elementos Essenciais

1. **Imports de Domain**: Entities e failures das camadas superiores
2. **Abstract Class**: Interface pura sem implementação
3. **Documentação de Protocolos**: Como a comunicação deve ocorrer
4. **Either Pattern**: Sempre `Either<Failure, Success>` para retornos
5. **Future Methods**: Operações assíncronas para I/O externo

---

## 📚 Exemplo Prático: IUserDatasource

### Interface Completa

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../../cogna_resale_core.dart' show Unit;
import '../../domain/entities/user_entity.dart';
import '../../domain/entities/user_notification_preferences_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/enums/account_person_type.dart';

/// Interface que define o contrato para comunicação com fontes de dados de usuários
/// 
/// Esta interface estabelece as operações de comunicação para:
/// - Acesso a APIs REST para gerenciamento de usuários
/// - Comunicação com banco de dados de usuários
/// - Operações de cache para dados de sessão
/// - Sincronização com sistemas de autenticação externos
abstract class IUserDatasource {
  /// Obtém o usuário atualmente logado via API/cache
  /// 
  /// Protocolo de comunicação:
  /// - Primeiro verificar cache local
  /// - Se não encontrado, consultar API de sessão
  /// - Atualizar cache com resultado
  /// 
  /// Retorna [Right] com [UserEntity] do usuário logado ou
  /// [Left] com:
  /// - [UserNotFoundError] se nenhum usuário logado
  /// - [UserSessionExpiredError] se token expirou
  /// - [UserServerError] para erros de comunicação
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();

  /// Obtém usuário específico por ID via API
  /// 
  /// [id] identificador único do usuário
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/{id}
  /// - Headers de autenticação obrigatórios
  /// - Timeout de 30 segundos
  /// 
  /// Retorna [Right] com [UserEntity] encontrado ou
  /// [Left] com:
  /// - [UserValidationError] se ID inválido
  /// - [UserNotFoundError] se usuário não existe (404)
  /// - [UserAuthorizationError] se não autorizado (401/403)
  /// - [UserServerError] para erros de comunicação (5xx)
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  });

  /// Cria um novo usuário via API
  /// 
  /// [data] dados do usuário em formato de entidade
  /// 
  /// Protocolo de comunicação:
  /// - POST /api/users
  /// - Content-Type: application/json
  /// - Body: JSON serializado da entidade
  /// - Headers de autenticação quando necessário
  /// 
  /// Retorna [Right] com [UserEntity] criado ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos (400)
  /// - [UserConflictError] se email/CPF já existe (409)
  /// - [UserServerError] para erros de servidor (5xx)
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  });

  /// Atualiza dados de um usuário via API
  /// 
  /// [data] dados atualizados do usuário (deve conter ID)
  /// 
  /// Protocolo de comunicação:
  /// - PUT /api/users/{id}
  /// - Content-Type: application/json
  /// - Body: JSON completo da entidade
  /// - Headers de autenticação obrigatórios
  /// 
  /// Retorna [Right] com [UserEntity] atualizado ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos (400)
  /// - [UserNotFoundError] se usuário não existe (404)
  /// - [UserConflictError] se conflito de dados (409)
  /// - [UserAuthorizationError] se não autorizado (401/403)
  /// - [UserServerError] para erros de servidor (5xx)
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });

  /// Remove um usuário via API
  /// 
  /// [id] identificador do usuário a ser removido
  /// 
  /// Protocolo de comunicação:
  /// - DELETE /api/users/{id}
  /// - Headers de autenticação obrigatórios
  /// - Confirmação via query parameter ?confirm=true
  /// 
  /// Retorna [Right] com [UserEntity] removido ou
  /// [Left] com:
  /// - [UserValidationError] se ID inválido
  /// - [UserNotFoundError] se usuário não existe (404)
  /// - [UserAuthorizationError] se não autorizado (401/403)
  /// - [UserBusinessRuleError] se violação de regras (422)
  /// - [UserServerError] para erros de servidor (5xx)
  Future<Either<IUserFailure, UserEntity>> deleteUser({
    required String id,
  });

  /// Remove a conta do usuário logado via API
  /// 
  /// Protocolo de comunicação:
  /// - DELETE /api/users/me
  /// - Headers de autenticação obrigatórios
  /// - Confirmação adicional via header X-Confirm-Delete
  /// 
  /// Retorna [Right] com [UserEntity] removido ou
  /// [Left] com erros específicos da operação
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount();

  /// Altera a senha de um usuário via API
  /// 
  /// [id] identificador do usuário
  /// [newPassword] nova senha (já criptografada)
  /// [currentPassword] senha atual para verificação
  /// 
  /// Protocolo de comunicação:
  /// - PATCH /api/users/{id}/password
  /// - Content-Type: application/json
  /// - Body: {"currentPassword": "...", "newPassword": "..."}
  /// - Headers de autenticação obrigatórios
  /// 
  /// Retorna [Right] com Unit se alterada com sucesso ou
  /// [Left] com:
  /// - [UserValidationError] se parâmetros inválidos (400)
  /// - [UserNotFoundError] se usuário não existe (404)
  /// - [UserAuthenticationError] se senha atual incorreta (401)
  /// - [UserServerError] para erros de servidor (5xx)
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });

  /// Busca usuários por critério textual via API
  /// 
  /// [query] termo de busca para nome/email
  /// [limit] máximo de resultados (padrão: 20, máximo: 100)
  /// [offset] deslocamento para paginação (padrão: 0)
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/search?q={query}&limit={limit}&offset={offset}
  /// - Headers de autenticação obrigatórios
  /// - Encoding UTF-8 para query strings
  /// 
  /// Retorna [Right] com lista de usuários encontrados ou
  /// [Left] com:
  /// - [UserValidationError] se query muito curta (400)
  /// - [UserServerError] para erros de busca (5xx)
  Future<Either<IUserFailure, List<UserEntity>>> searchUsers({
    required String query,
    int? limit,
    int? offset,
  });

  /// Obtém usuários filtrados por tipo via API
  /// 
  /// [personType] tipo de pessoa a filtrar
  /// [page] página desejada (padrão: 1)
  /// [limit] itens por página (padrão: 20, máximo: 100)
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users?personType={type}&page={page}&limit={limit}
  /// - Headers de autenticação obrigatórios
  /// 
  /// Retorna [Right] com lista paginada ou
  /// [Left] com erros de consulta
  Future<Either<IUserFailure, List<UserEntity>>> getUsersByPersonType({
    required AccountPersonType personType,
    int? page,
    int? limit,
  });

  /// Verifica disponibilidade de email via API
  /// 
  /// [email] email a ser validado
  /// [excludeUserId] ID do usuário a excluir da verificação
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/email-available?email={email}&exclude={id}
  /// - Headers de autenticação opcionais
  /// - Response: {"available": true/false}
  /// 
  /// Retorna [Right] com true se disponível ou
  /// [Left] com erro na verificação
  Future<Either<IUserFailure, bool>> isEmailAvailable({
    required String email,
    String? excludeUserId,
  });

  /// Verifica disponibilidade de CPF via API
  /// 
  /// [cpf] CPF a ser validado (apenas números)
  /// [excludeUserId] ID do usuário a excluir da verificação
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/cpf-available?cpf={cpf}&exclude={id}
  /// - Headers de autenticação opcionais
  /// - Response: {"available": true/false}
  /// 
  /// Retorna [Right] com true se disponível ou
  /// [Left] com erro na verificação
  Future<Either<IUserFailure, bool>> isCpfAvailable({
    required String cpf,
    String? excludeUserId,
  });

  /// Obtém contagem total de usuários via API
  /// 
  /// [personType] filtrar por tipo específico (opcional)
  /// [activeOnly] contar apenas usuários ativos (padrão: true)
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/count?personType={type}&activeOnly={bool}
  /// - Headers de autenticação obrigatórios
  /// - Response: {"count": number}
  /// 
  /// Retorna [Right] com total de usuários ou
  /// [Left] com erro de consulta
  Future<Either<IUserFailure, int>> getUsersCount({
    AccountPersonType? personType,
    bool activeOnly = true,
  });

  /// Atualiza preferências de notificação via API
  /// 
  /// [userId] identificador do usuário
  /// [preferences] novas preferências de notificação
  /// 
  /// Protocolo de comunicação:
  /// - PATCH /api/users/{id}/notification-preferences
  /// - Content-Type: application/json
  /// - Body: JSON das preferências
  /// 
  /// Retorna [Right] com preferências atualizadas ou
  /// [Left] com erro na atualização
  Future<Either<IUserFailure, UserNotificationPreferencesEntity>> updateUserNotificationPreferences({
    required String userId,
    required UserNotificationPreferencesEntity preferences,
  });

  /// Obtém usuários por período de criação via API
  /// 
  /// [startDate] data inicial do período
  /// [endDate] data final do período
  /// [limit] máximo de resultados (opcional)
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/by-date?start={iso_date}&end={iso_date}&limit={limit}
  /// - Headers de autenticação obrigatórios
  /// - Datas em formato ISO 8601
  /// 
  /// Retorna [Right] com lista de usuários ou
  /// [Left] com erro na consulta
  Future<Either<IUserFailure, List<UserEntity>>> getUsersByDateRange({
    required DateTime startDate,
    required DateTime endDate,
    int? limit,
  });

  /// Sincroniza dados de usuário com sistemas externos
  /// 
  /// [userId] identificador do usuário a sincronizar
  /// [systems] lista de sistemas externos para sincronizar
  /// 
  /// Protocolo de comunicação:
  /// - POST /api/users/{id}/sync
  /// - Content-Type: application/json
  /// - Body: {"systems": ["system1", "system2"]}
  /// 
  /// Retorna [Right] com Unit se sincronizado ou
  /// [Left] com erro na sincronização
  Future<Either<IUserFailure, Unit>> syncUserWithExternalSystems({
    required String userId,
    required List<String> systems,
  });

  /// Obtém dados de auditoria de um usuário via API
  /// 
  /// [userId] identificador do usuário
  /// [limit] máximo de registros de auditoria
  /// 
  /// Protocolo de comunicação:
  /// - GET /api/users/{id}/audit?limit={limit}
  /// - Headers de autenticação obrigatórios
  /// - Requer permissões especiais
  /// 
  /// Retorna [Right] com dados de auditoria ou
  /// [Left] com erro de acesso
  Future<Either<IUserFailure, List<Map<String, dynamic>>>> getUserAuditData({
    required String userId,
    int? limit,
  });
}
```

---

## 📋 Template para Interfaces DataSource

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';

/// Interface que define o contrato para comunicação com fontes de dados de [Entity]
/// 
/// Esta interface estabelece as operações de comunicação para:
/// - [tipo de fonte 1] (ex: API REST)
/// - [tipo de fonte 2] (ex: banco de dados)
/// - [tipo de fonte 3] (ex: cache/redis)
abstract class I[Entity]Datasource {
  /// [Breve descrição da operação de comunicação]
  /// 
  /// [param] - descrição do parâmetro
  /// 
  /// Protocolo de comunicação:
  /// - [método HTTP] [endpoint]
  /// - [headers necessários]
  /// - [formato de dados]
  /// 
  /// Retorna [Right] com [tipo de retorno] ou
  /// [Left] com:
  /// - [TipoError] para [condição de erro]
  /// - [OutroTipoError] para [outra condição]
  Future<Either<I[Entity]Failure, [ReturnType]>> [methodName]({
    required [Type] [param],
    [Type]? [optionalParam],
  });
}
```

### Operações de Comunicação Padrão

```dart
abstract class I[Entity]Datasource {
  // CREATE - Criar via API/BD
  Future<Either<I[Entity]Failure, [Entity]Entity>> create[Entity]({
    required [Entity]Entity data,
  });

  // READ - Buscar via API/BD
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]ById({
    required String id,
  });

  Future<Either<I[Entity]Failure, List<[Entity]Entity>>> getAll[Entity]s({
    int? limit,
    int? offset,
  });

  // UPDATE - Atualizar via API/BD
  Future<Either<I[Entity]Failure, [Entity]Entity>> update[Entity]({
    required [Entity]Entity data,
  });

  // DELETE - Remover via API/BD
  Future<Either<I[Entity]Failure, [Entity]Entity>> delete[Entity]({
    required String id,
  });

  // SEARCH - Buscar com critérios
  Future<Either<I[Entity]Failure, List<[Entity]Entity>>> search[Entity]s({
    required String query,
    int? limit,
  });

  // CACHE - Operações de cache quando aplicável
  Future<Either<I[Entity]Failure, [Entity]Entity?>> get[Entity]FromCache({
    required String key,
  });

  Future<Either<I[Entity]Failure, Unit>> set[Entity]InCache({
    required String key,
    required [Entity]Entity data,
    Duration? ttl,
  });

  // SYNC - Sincronização com sistemas externos
  Future<Either<I[Entity]Failure, Unit>> sync[Entity]WithExternalSystem({
    required String [entity]Id,
    required String systemId,
  });
}
```

### Convenções de Interface

**Nomenclatura:**
- Interface: `I[Entity]Datasource`
- Métodos de API: `get[Entity]ById`, `create[Entity]`, `update[Entity]`
- Métodos de cache: `get[Entity]FromCache`, `set[Entity]InCache`
- Métodos de sync: `sync[Entity]WithExternalSystem`

**Documentação de Protocolos:**
- Especificar método HTTP e endpoint
- Detalhar headers obrigatórios e opcionais
- Documentar formato de dados (JSON, XML, etc.)
- Mapear códigos de status HTTP para tipos de erro
- Especificar timeouts e retry policies quando relevante

**Padrões de Comunicação:**
- Sempre `Future<Either<IFailure, Success>>`
- Métodos assíncronos para todas as operações de I/O
- Parâmetros opcionais para paginação e filtros
- Documentação clara de protocolos de comunicação

---

## 📋 Checklist para Interfaces DataSource

### Checklist de Criação ✅

**Estrutura da Interface:**
- [ ] Localizada em `lib/src/infra/datasources/`
- [ ] Nome seguindo padrão `I[Entity]Datasource`
- [ ] Declarada como `abstract class`
- [ ] Imports de entities e failures do domain
- [ ] Sem implementação de métodos

**Operações de Comunicação:**
- [ ] Métodos para operações CRUD via protocolo externo
- [ ] Métodos de busca e filtros
- [ ] Operações de cache quando aplicável
- [ ] Sincronização com sistemas externos quando necessário
- [ ] Métodos de validação remota (disponibilidade, etc.)

**Documentação de Protocolos:**
- [ ] Especificação de métodos HTTP para APIs REST
- [ ] Documentação de endpoints e estrutura de URLs
- [ ] Detalhamento de headers obrigatórios e opcionais
- [ ] Formato de dados (JSON, XML, binário)
- [ ] Mapeamento de códigos de status para tipos de erro
- [ ] Timeouts e políticas de retry quando relevante

**Tratamento de Erros:**
- [ ] Mapeamento claro de erros de comunicação
- [ ] Diferenciação entre erros de rede e de aplicação
- [ ] Tratamento de timeouts e indisponibilidade
- [ ] Erros de autenticação e autorização
- [ ] Erros de validação de dados

**Padrões de Retorno:**
- [ ] Sempre `Future<Either<IFailure, Success>>`
- [ ] Consistência nos tipos de erro entre métodos
- [ ] Parâmetros opcionais para configuração de comunicação
- [ ] Documentação de todos os cenários de erro possíveis

---

## 🎯 Diretrizes para Interfaces

### ✅ Boas Práticas

```dart
// ✅ Interface bem documentada com protocolos claros
/// Interface para comunicação com API de produtos
/// 
/// Protocolo base: REST API sobre HTTPS
/// Base URL: https://api.exemplo.com/v1
/// Autenticação: Bearer Token via header Authorization
abstract class IProductDatasource {
  /// Obtém produto por ID via API
  /// 
  /// Protocolo:
  /// - GET /api/products/{id}
  /// - Headers: Authorization: Bearer {token}
  /// - Timeout: 30 segundos
  /// 
  /// Retorna [Right] com produto ou [Left] com erro
  Future<Either<IProductFailure, ProductEntity>> getProductById({
    required String id,
  });
}

// ✅ Parâmetros de comunicação bem definidos
Future<Either<IProductFailure, List<ProductEntity>>> searchProducts({
  required String query,
  int limit = 20,
  int offset = 0,
  List<String>? categories,
  Map<String, String>? additionalHeaders,
});

// ✅ Operações de cache explícitas
Future<Either<IProductFailure, ProductEntity?>> getProductFromCache({
  required String productId,
});

Future<Either<IProductFailure, Unit>> setProductInCache({
  required String productId,
  required ProductEntity product,
  Duration ttl = const Duration(minutes: 15),
});
```

### ❌ Evitar

```dart
// ❌ Interface sem documentação de protocolo
abstract class IProductDatasource {
  Future<Either<IProductFailure, ProductEntity>> getData();
}

// ❌ Métodos que misturam fontes de dados
Future<Either<IProductFailure, ProductEntity>> getProductFromApiOrCache();

// ❌ Parâmetros de configuração de infraestrutura
Future<Either<IProductFailure, ProductEntity>> getProduct({
  required String id,
  required String apiKey, // configuração, não parâmetro de negócio
  required Duration timeout, // configuração de infraestrutura
});

// ❌ Retornos sem Either
Future<ProductEntity> getProduct(); // pode falhar sem tratamento

// ❌ Dependências de implementação específica
import 'package:dio/dio.dart'; // não deve depender de lib específica
```

---

## 🚀 Exemplo de Uso da Interface

```dart
// Na camada de data (implementação)
class UserDatasource extends IUserDatasource {
  UserDatasource({
    required this.httpClient,
    required this.cacheService,
  });
  
  final HttpClient httpClient;
  final CacheService cacheService;

  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    try {
      final response = await httpClient.get('/api/users/$id');
      final userData = UserEntity.fromJson(response.data);
      return Right(userData);
    } on NetworkException catch (e) {
      return Left(UserServerError(message: e.message));
    } on ValidationException catch (e) {
      return Left(UserValidationError(message: e.message));
    }
  }
}

// Na camada de infrastructure (repository)
class UserRepository extends IUserRepository {
  UserRepository({required this.datasource});
  
  final IUserDatasource datasource; // Dependendo da interface

  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    // Delegando para o datasource
    return datasource.getUserById(id: id);
  }
}
```

Esta estrutura garante que as interfaces de datasources sejam bem definidas e estabeleçam contratos claros para comunicação com fontes de dados externas, mantendo a separação de responsabilidades da arquitetura limpa.
