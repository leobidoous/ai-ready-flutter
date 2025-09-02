# Domain Repositories (Interfaces) - Clean Architecture

## 📚 Visão Geral

Esta documentação define as **interfaces de repositórios** da camada de **Domain** em nossa arquitetura limpa. As interfaces estabelecem **O QUE** deve ser feito em termos de acesso aos dados, definindo contratos puros sem se preocupar com **COMO** implementar.

### 🎯 Princípios Fundamentais das Interfaces Repository

**O QUE as interfaces DEFINEM:**
- ✅ **Contratos de Persistência**: QUE operações de dados devem existir
- ✅ **Assinaturas Puras**: Métodos sem implementação, apenas contratos
- ✅ **Operações CRUD**: Create, Read, Update, Delete tipados fortemente
- ✅ **Tratamento de Erros**: Either pattern obrigatório para todas as operações
- ✅ **Validações de Negócio**: Documentação das regras aplicáveis

**O QUE as interfaces NÃO FAZEM:**
- ❌ **Não implementam acesso real**: Apenas definem contratos
- ❌ **Não dependem de tecnologias**: Não especificam DB, HTTP, cache
- ❌ **Não contêm lógica de infraestrutura**: Zero detalhes de implementação
- ❌ **Não importam dependências externas**: Apenas entities e failures do domain
- ❌ **Não quebram SOLID**: Dependem apenas de abstrações, nunca implementações

```

---

## 🔒 Princípios SOLID em Interfaces Repository

### 1. **Dependency Inversion Principle (DIP)**
```dart
// ✅ Interface depende apenas de abstrações do domain
import '../entities/user_entity.dart';        // Abstração
import '../failures/i_user_failures.dart';    // Abstração

abstract class IUserRepository {
  // Métodos dependem apenas de entities e failures (abstrações)
  Future<Either<IUserFailure, UserEntity>> getUser();
}

// ❌ NUNCA depender de implementações concretas
// import 'package:dio/dio.dart';                    // Implementação concreta
// import '../infra/datasources/user_datasource.dart'; // Implementação concreta
```

### 2. **Interface Segregation Principle (ISP)**
```dart
// ✅ Interface específica por responsabilidade
abstract class IUserRepository {
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity data});
}

// ✅ Separar interfaces por contextos diferentes
abstract class IUserCacheRepository {
  Future<Either<IUserFailure, UserEntity?>> getUserFromCache();
  Future<Either<IUserFailure, Unit>> saveUserToCache({required UserEntity user});
}

// ❌ Interface única com responsabilidades misturadas
abstract class IUserEverythingRepository {
  // user operations + cache + auth + notifications... EVITAR!
}
```

### 3. **Tipagem Forte e Either Pattern Obrigatório**
```dart
// ✅ Tipagem forte com Either para TODAS as operações
abstract class IUserRepository {
  /// SEMPRE Either<Failure, Success> para operações que podem falhar
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id, // Tipagem forte e obrigatória
  });
  
  /// Unit para operações sem retorno específico
  Future<Either<IUserFailure, Unit>> deleteUser({
    required String id,
  });
  
  /// Listas também podem falhar
  Future<Either<IUserFailure, List<UserEntity>>> getUsers({
    int? page,
    int? limit,
  });
}

// ❌ Evitar operações sem tratamento de erro
// Future<UserEntity> getUser();        // Pode falhar sem Either
// UserEntity getUserSync();            // Operações síncronas para I/O  
// Future<bool> deleteUser();           // Retorno genérico demais
// List<UserEntity> getUsers();         // Sem tratamento de erro
```

---

## 🏗️ Estrutura de Arquivos

---

## 🔍 Anatomia de uma Interface Repository

### Componentes Principais

```dart
import 'package:base_core/base_core.dart' show Either;
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

/// Interface que define o contrato para persistência de dados de usuários
/// 
/// Esta interface estabelece as operações de acesso aos dados para:
/// - Operações CRUD básicas (Create, Read, Update, Delete)
/// - Consultas específicas por critérios
/// - Validações de unicidade
/// - Operações de autenticação e sessão
abstract class IUserRepository {
  // Métodos definindo contratos de acesso aos dados
}
```

### Elementos Essenciais

1. **Imports Restritos**: Apenas entities e failures do domain
2. **Abstract Class**: Interface pura sem implementação
3. **Documentação Completa**: Contratos claros e bem documentados
4. **Either Pattern**: Sempre `Either<Failure, Success>` para retornos
5. **Future Methods**: Operações assíncronas para I/O

---

## 📚 Exemplo Prático: IUserRepository

### Interface Completa

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../../cogna_resale_core.dart' show Unit;
import '../entities/user_entity.dart';
import '../entities/user_notification_preferences_entity.dart';
import '../failures/i_user_failures.dart';
import '../enums/account_person_type.dart';

/// Interface que define o contrato para persistência de dados de usuários
/// 
/// Esta interface estabelece as operações de acesso aos dados para:
/// - Operações CRUD completas (Create, Read, Update, Delete)
/// - Consultas por critérios específicos
/// - Validações de unicidade e integridade
/// - Gerenciamento de sessão e autenticação
abstract class IUserRepository {
  /// Obtém o usuário atualmente logado no sistema
  /// 
  /// Retorna [Right] com [UserEntity] do usuário logado ou
  /// [Left] com:
  /// - [UserNotFoundError] se nenhum usuário logado
  /// - [UserSessionExpiredError] se sessão expirou
  /// - [UserServerError] para erros de acesso aos dados
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();

  /// Obtém usuário específico por ID
  /// 
  /// [id] identificador único do usuário (não pode ser vazio)
  /// 
  /// Retorna [Right] com [UserEntity] encontrado ou
  /// [Left] com:
  /// - [UserValidationError] se ID inválido
  /// - [UserNotFoundError] se usuário não existe
  /// - [UserServerError] para erros de acesso aos dados
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  });

  /// Cria um novo usuário no sistema
  /// 
  /// [data] dados do usuário a ser criado (deve ter dados válidos)
  /// 
  /// Validações aplicadas:
  /// - Email único no sistema
  /// - CPF único no sistema
  /// - Dados obrigatórios preenchidos
  /// 
  /// Retorna [Right] com [UserEntity] criado ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos
  /// - [UserConflictError] se email/CPF já existe
  /// - [UserServerError] para erros de persistência
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  });

  /// Atualiza dados de um usuário existente
  /// 
  /// [data] dados atualizados do usuário (deve conter ID válido)
  /// 
  /// Validações aplicadas:
  /// - Usuário deve existir
  /// - Email único (exceto para o próprio usuário)
  /// - CPF único (exceto para o próprio usuário)
  /// 
  /// Retorna [Right] com [UserEntity] atualizado ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos
  /// - [UserNotFoundError] se usuário não existe
  /// - [UserConflictError] se email/CPF já existe
  /// - [UserServerError] para erros de persistência
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });

  /// Remove um usuário do sistema
  /// 
  /// [id] identificador do usuário a ser removido
  /// 
  /// ⚠️ OPERAÇÃO IRREVERSÍVEL ⚠️
  /// Remove permanentemente o usuário e todos os dados relacionados
  /// 
  /// Retorna [Right] com [UserEntity] removido ou
  /// [Left] com:
  /// - [UserValidationError] se ID inválido
  /// - [UserNotFoundError] se usuário não existe
  /// - [UserBusinessRuleError] se violação de regras
  /// - [UserServerError] para erros de persistência
  Future<Either<IUserFailure, UserEntity>> deleteUser({
    required String id,
  });

  /// Remove completamente a conta do usuário logado
  /// 
  /// Operação específica para auto-exclusão, aplicando regras 
  /// de negócio específicas para conta própria
  /// 
  /// Retorna [Right] com [UserEntity] removido ou
  /// [Left] com:
  /// - [UserNotFoundError] se nenhum usuário logado
  /// - [UserBusinessRuleError] se não pode remover conta própria
  /// - [UserServerError] para erros de persistência
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount();

  /// Altera a senha de um usuário
  /// 
  /// [id] identificador do usuário
  /// [newPassword] nova senha (já criptografada)
  /// [currentPassword] senha atual para verificação
  /// 
  /// Validações aplicadas:
  /// - Verificação da senha atual
  /// - Nova senha deve ser diferente da atual
  /// 
  /// Retorna [Right] com Unit se alterada com sucesso ou
  /// [Left] com:
  /// - [UserValidationError] se parâmetros inválidos
  /// - [UserNotFoundError] se usuário não existe
  /// - [UserAuthenticationError] se senha atual incorreta
  /// - [UserServerError] para erros de persistência
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });

  /// Busca usuários por critério textual
  /// 
  /// [query] termo de busca (nome, email) - mínimo 3 caracteres
  /// [limit] máximo de resultados (padrão: 20, máximo: 100)
  /// [offset] deslocamento para paginação (padrão: 0)
  /// 
  /// Retorna [Right] com lista de usuários encontrados ou
  /// [Left] com:
  /// - [UserValidationError] se query muito curta
  /// - [UserServerError] para erros de consulta
  Future<Either<IUserFailure, List<UserEntity>>> searchUsers({
    required String query,
    int? limit,
    int? offset,
  });

  /// Obtém usuários filtrados por tipo de pessoa
  /// 
  /// [personType] tipo de pessoa a filtrar
  /// [page] página desejada (padrão: 1)
  /// [limit] itens por página (padrão: 20, máximo: 100)
  /// 
  /// Retorna [Right] com lista paginada ou
  /// [Left] com:
  /// - [UserValidationError] se parâmetros inválidos
  /// - [UserServerError] para erros de consulta
  Future<Either<IUserFailure, List<UserEntity>>> getUsersByPersonType({
    required AccountPersonType personType,
    int? page,
    int? limit,
  });

  /// Valida se email está disponível para uso
  /// 
  /// [email] email a ser validado
  /// [excludeUserId] ID do usuário a excluir da verificação (para updates)
  /// 
  /// Retorna [Right] com true se disponível ou
  /// [Left] com:
  /// - [UserValidationError] se email inválido
  /// - [UserServerError] para erros de consulta
  Future<Either<IUserFailure, bool>> isEmailAvailable({
    required String email,
    String? excludeUserId,
  });

  /// Valida se CPF está disponível para uso
  /// 
  /// [cpf] CPF a ser validado (apenas números)
  /// [excludeUserId] ID do usuário a excluir da verificação
  /// 
  /// Retorna [Right] com true se disponível ou
  /// [Left] com:
  /// - [UserValidationError] se CPF inválido
  /// - [UserServerError] para erros de consulta
  Future<Either<IUserFailure, bool>> isCpfAvailable({
    required String cpf,
    String? excludeUserId,
  });

  /// Obtém o total de usuários cadastrados
  /// 
  /// [personType] filtrar por tipo específico (opcional)
  /// [activeOnly] contar apenas usuários ativos (padrão: true)
  /// 
  /// Retorna [Right] com total de usuários ou
  /// [Left] com erro de consulta
  Future<Either<IUserFailure, int>> getUsersCount({
    AccountPersonType? personType,
    bool activeOnly = true,
  });

  /// Atualiza as preferências de notificação do usuário
  /// 
  /// [userId] identificador do usuário
  /// [preferences] novas preferências de notificação
  /// 
  /// Retorna [Right] com preferências atualizadas ou
  /// [Left] com erro na atualização
  Future<Either<IUserFailure, UserNotificationPreferencesEntity>> updateUserNotificationPreferences({
    required String userId,
    required UserNotificationPreferencesEntity preferences,
  });

  /// Obtém usuários criados em um período específico
  /// 
  /// [startDate] data inicial do período
  /// [endDate] data final do período
  /// [limit] máximo de resultados (opcional)
  /// 
  /// Retorna [Right] com lista de usuários ou
  /// [Left] com erro na consulta
  Future<Either<IUserFailure, List<UserEntity>>> getUsersByDateRange({
    required DateTime startDate,
    required DateTime endDate,
    int? limit,
  });
}
```

---

## 📋 Template para Interfaces Repository

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show Either;
import '../entities/[entity]_entity.dart';
import '../failures/i_[entity]_failures.dart';

/// Interface que define o contrato para persistência de dados de [Entity]
/// 
/// Esta interface estabelece as operações de acesso aos dados para:
/// - [operação 1]
/// - [operação 2]
/// - [operação N]
abstract class I[Entity]Repository {
  /// [Breve descrição da operação de acesso aos dados]
  /// 
  /// [param] - descrição do parâmetro (obrigatório/opcional)
  /// 
  /// Validações aplicadas:
  /// - [validação 1]
  /// - [validação 2]
  /// 
  /// Retorna [Right] com [tipo de retorno] ou
  /// [Left] com:
  /// - [TipoError] em caso de [condição]
  /// - [OutroTipoError] em caso de [outra condição]
  Future<Either<I[Entity]Failure, [ReturnType]>> [methodName]({
    required [Type] [param],
    [Type]? [optionalParam],
  });
}
```

### Operações CRUD Padrão

```dart
abstract class I[Entity]Repository {
  // CREATE
  Future<Either<I[Entity]Failure, [Entity]Entity>> create[Entity]({
    required [Entity]Entity data,
  });

  // READ
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]ById({
    required String id,
  });

  Future<Either<I[Entity]Failure, List<[Entity]Entity>>> getAll[Entity]s({
    int? limit,
    int? offset,
  });

  // UPDATE
  Future<Either<I[Entity]Failure, [Entity]Entity>> update[Entity]({
    required [Entity]Entity data,
  });

  // DELETE
  Future<Either<I[Entity]Failure, [Entity]Entity>> delete[Entity]({
    required String id,
  });

  // SEARCH
  Future<Either<I[Entity]Failure, List<[Entity]Entity>>> search[Entity]s({
    required String query,
    int? limit,
  });

  // COUNT
  Future<Either<I[Entity]Failure, int>> get[Entity]sCount();
}
```

### Convenções de Interface

**Nomenclatura:**
- Interface: `I[Entity]Repository`
- Métodos CRUD: `create[Entity]`, `get[Entity]ById`, `update[Entity]`, `delete[Entity]`
- Métodos de busca: `search[Entity]s`, `get[Entity]sByType`
- Métodos de contagem: `get[Entity]sCount`

**Documentação Obrigatória:**
- Descrição geral da interface
- Propósito de cada operação
- Parâmetros e sua obrigatoriedade
- Validações aplicadas no acesso aos dados
- Mapeamento completo de retornos possíveis

**Padrões de Retorno:**
- Sempre `Future<Either<IFailure, Success>>`
- Parâmetros nomeados com `required` quando obrigatórios
- Consistência nos tipos de erro entre métodos similares

---

## 📋 Checklist para Interfaces Repository

### Checklist de Criação ✅

**Estrutura da Interface:**
- [ ] Localizada em `lib/src/domain/repositories/`
- [ ] Nome seguindo padrão `I[Entity]Repository`
- [ ] Declarada como `abstract class`
- [ ] Imports apenas de entities e failures do domain
- [ ] Sem implementação de métodos

**Operações CRUD:**
- [ ] Método para criação (`create[Entity]`)
- [ ] Método para leitura por ID (`get[Entity]ById`)
- [ ] Método para atualização (`update[Entity]`)
- [ ] Método para exclusão (`delete[Entity]`)
- [ ] Métodos de listagem com paginação

**Métodos de Consulta:**
- [ ] Busca textual (`search[Entity]s`)
- [ ] Filtros por critérios específicos
- [ ] Contadores (`get[Entity]sCount`)
- [ ] Validações de unicidade quando aplicável

**Documentação Obrigatória:**
- [ ] Descrição geral da interface e responsabilidades
- [ ] Documentação de cada método com propósito claro
- [ ] Especificação de todos os parâmetros
- [ ] Listagem de validações aplicadas no acesso aos dados
- [ ] Mapeamento completo de tipos de erro possíveis

**Padrões de Retorno:**
- [ ] Sempre `Future<Either<IFailure, Success>>`
- [ ] Métodos assíncronos para operações de I/O
- [ ] Consistência nos tipos de erro entre métodos
- [ ] Parâmetros opcionais com valores padrão quando adequado

---

## 🎯 Diretrizes para Interfaces

### ✅ Boas Práticas

```dart
// ✅ Interface bem documentada com propósito claro
/// Interface que define o contrato para persistência de dados de produtos
/// 
/// Responsável por estabelecer contratos para:
/// - Operações CRUD completas
/// - Consultas por categorias e filtros
/// - Validações de disponibilidade
abstract class IProductRepository {
  /// Obtém produto por ID específico
  /// 
  /// [id] identificador único do produto
  /// 
  /// Retorna [Right] com produto encontrado ou
  /// [Left] com [ProductNotFoundError] se não existe
  Future<Either<IProductFailure, ProductEntity>> getProductById({
    required String id,
  });
}

// ✅ Parâmetros bem especificados
Future<Either<IProductFailure, List<ProductEntity>>> getProductsByCategory({
  required String categoryId,
  int limit = 20,
  int offset = 0,
  bool activeOnly = true,
});

// ✅ Métodos com propósito único e claro
Future<Either<IProductFailure, bool>> isProductCodeAvailable({
  required String code,
  String? excludeProductId,
});
```

### ❌ Evitar

```dart
// ❌ Interface sem documentação
abstract class IProductRepository {
  Future<Either<IProductFailure, ProductEntity>> doSomething();
}

// ❌ Métodos com múltiplas responsabilidades
Future<Either<IProductFailure, ProductEntity>> createProductAndNotifyUsers();

// ❌ Retornos inconsistentes
Future<ProductEntity> getProduct(); // sem Either
Either<IProductFailure, ProductEntity> getProductSync(); // sem Future

// ❌ Parâmetros posicionais obrigatórios
Future<Either<IProductFailure, ProductEntity>> updateProduct(ProductEntity data);

// ❌ Dependências de infraestrutura
import 'package:dio/dio.dart'; // não deve importar dependências externas
```

---

## 🚀 Exemplo de Uso da Interface

```dart
// Na camada de infrastructure (implementação)
class UserRepository extends IUserRepository {
  UserRepository({required this.datasource});
  
  final IUserDatasource datasource;

  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    // Implementação delegando para datasource
    return datasource.getUserById(id: id);
  }
}

// Na camada de domain (use case)
class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});
  
  final IUserRepository repository; // Dependendo da interface, não da implementação

  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    // Usando a interface repository
    return repository.getUserById(id: id);
  }
}
```

Esta estrutura garante que as interfaces de repositórios sejam bem definidas e mantenham a separação clara entre as camadas do Clean Architecture, estabelecendo contratos claros para acesso aos dados sem se preocupar com detalhes de implementação.
