# Entities - Domain Layer

## 📋 O que são Entities?

**Entities** são objetos de negócio puros que representam conceitos fundamentais do domínio da aplicação. Elas carregam dados e regras de negócio básicas, mantendo-se completamente independentes de frameworks, bibliotecas externas ou detalhes de implementação.

### 🎯 Responsabilidades

**✅ O que Entities FAZEM:**

- Representam **conceitos de negócio** (User, Address, Product, etc.)
- Carregam **dados estruturados** com tipagem forte
- Implementam **imutabilidade** através de `const` constructors
- Fornecem método **copyWith** para criar cópias modificadas
- Definem **validações básicas** através de `assert` (quando necessário)
- Compõem outras entities para criar **objetos complexos**

**❌ O que Entities NÃO FAZEM:**

- Não contêm lógica de serialização/deserialização (fica nos Models)
- Não dependem de frameworks externos (Dio, HTTP, etc.)
- Não implementam comunicação com APIs ou databases
- Não contêm lógica de apresentação ou formatação de UI
- Não herdam de classes externas (exceto quando necessário para composição)

---

## 🏗️ Exemplo Completo: UserEntity

Este exemplo único demonstra **todos os cenários comuns** que você encontrará:

```dart
import '../enums/user_status.dart';
import '../enums/user_gender_type.dart';
import 'address_entity.dart';

class UserEntity {
  const UserEntity({
    required this.id,
    required this.name,
    required this.email,
    required this.cpf,
    required this.birth,
    required this.status,
    required this.gender,
    required this.isActive,
    required this.score,
    required this.address,
    required this.tags,
    this.phone,
    this.lastLoginAt,
  });

  // 1. String - Dados textuais simples
  final String id;
  final String name;
  final String email;
  final String cpf;

  // 2. String? - Dados opcionais (nullable)
  final String? phone;

  // 3. DateTime - Datas obrigatórias
  final DateTime birth;

  // 4. DateTime? - Datas opcionais
  final DateTime? lastLoginAt;

  // 5. Enum - Valores constantes tipados
  final UserStatus status;          // active, inactive, blocked
  final UserGenderType gender;      // male, female, other

  // 6. bool - Flags booleanas
  final bool isActive;

  // 7. int/double - Valores numéricos
  final double score;

  // 8. Entity - Composição com outras entities
  final AddressEntity address;

  // 9. List - Coleções de dados
  final List<String> tags;

  UserEntity copyWith({
    String? id,
    String? name,
    String? email,
    String? cpf,
    String? phone,
    DateTime? birth,
    DateTime? lastLoginAt,
    UserStatus? status,
    UserGenderType? gender,
    bool? isActive,
    double? score,
    AddressEntity? address,
    List<String>? tags,
  }) {
    return UserEntity(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      cpf: cpf ?? this.cpf,
      phone: phone ?? this.phone,
      birth: birth ?? this.birth,
      lastLoginAt: lastLoginAt ?? this.lastLoginAt,
      status: status ?? this.status,
      gender: gender ?? this.gender,
      isActive: isActive ?? this.isActive,
      score: score ?? this.score,
      address: address ?? this.address,
      tags: tags ?? this.tags,
    );
  }
}
```

### 🔑 Elementos Essenciais

1. **Constructor `const`**: Permite imutabilidade e otimização de memória
2. **Campos `final`**: Garante que os dados não sejam modificados após criação
3. **Named parameters `required`**: Torna explícito quais dados são obrigatórios
4. **Nullable (`?`)**: Para campos opcionais que podem ser nulos
5. **Método `copyWith`**: Permite criar cópias modificadas mantendo imutabilidade
6. **Tipagem forte**: Usa tipos específicos (Enums, Entities, DateTime, bool, etc.)

---

## 🎯 Tipos de Dados Comuns

### 1. **String** - Dados Textuais

```dart
final String id;
final String name;
final String email;
```

**Use para:** IDs, nomes, emails, CPF, telefones, descrições.

### 2. **String?** - Dados Opcionais

```dart
final String? phone;
final String? middleName;
```

**Use para:** Campos que podem não existir (telefone opcional, nome do meio, etc.).

### 3. **DateTime** - Datas Obrigatórias

```dart
final DateTime birth;
final DateTime createdAt;
```

**Use para:** Datas de nascimento, criação, atualização.

### 4. **DateTime?** - Datas Opcionais

```dart
final DateTime? lastLoginAt;
final DateTime? deletedAt;
```

**Use para:** Último login, data de exclusão, datas que podem não existir.

### 5. **Enum** - Valores Constantes

```dart
final UserStatus status;        // active, inactive, blocked
final UserGenderType gender;    // male, female, other
```

**Use para:** Status, tipos, categorias - evita "strings mágicas".

### 6. **bool** - Flags Booleanas

```dart
final bool isActive;
final bool isVerified;
final bool hasAcceptedTerms;
```

**Use para:** Flags de estado (ativo/inativo, verificado/não verificado).

### 7. **int / double** - Valores Numéricos

```dart
final int age;
final double score;
final double balance;
```

**Use para:** Idades, pontuações, valores monetários, contadores.

### 8. **Entity** - Composição

```dart
final AddressEntity address;
final CompanyEntity company;
```

**Use para:** Objetos complexos que são entities por si só.

### 9. **List** - Coleções

```dart
final List<String> tags;
final List<String> permissions;
final List<OrderEntity> orders;
```

**Use para:** Múltiplos valores (tags, permissões, lista de pedidos).

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo            | Padrão                      | Exemplo                                   |
| --------------- | --------------------------- | ----------------------------------------- |
| **Classe**      | `{Nome}Entity`              | `UserEntity`, `AddressEntity`             |
| **Arquivo**     | `{nome}_entity.dart`        | `user_entity.dart`, `address_entity.dart` |
| **Campos**      | `camelCase`                 | `firstName`, `birthDate`, `isActive`      |
| **Constructor** | `const {Nome}Entity({...})` | `const UserEntity({...})`                 |

### ✅ Organização de Imports

```dart
// 1. Enums do domain
import '../enums/user_status.dart';
import '../enums/user_gender_type.dart';

// 2. Outras entities do domain
import 'address_entity.dart';
import 'company_entity.dart';

// ❌ NUNCA importar de outras camadas
// import '../../infra/models/user_model.dart';  // ERRADO!
// import 'package:dio/dio.dart';                 // ERRADO!
```

### ✅ Ordem dos Elementos na Classe

```dart
class UserEntity {
  // 1. Constructor
  const UserEntity({...});

  // 2. Campos (agrupados por tipo/contexto)
  final String id;
  final String name;

  final UserStatus status;
  final UserGenderType gender;

  final AddressEntity address;

  // 3. Método copyWith
  UserEntity copyWith({...}) {...}

  // 4. Getters/métodos auxiliares (se necessário)
  // bool get isActive => status == UserStatus.active;
}
```

---

## 🔧 Validações (Quando Usar)

### ✅ Validações Simples com Assert

```dart
class UserEntity {
  const UserEntity({
    required this.id,
    required this.age,
  }) : assert(age >= 0, 'Age must be non-negative'),
       assert(id.length > 0, 'ID cannot be empty');

  final String id;
  final int age;
}
```

**Use para:** Validações simples que garantem consistência básica dos dados.

### ❌ Validações Complexas (Evitar)

```dart
// ❌ EVITAR: Validações complexas na Entity
class UserEntity {
  const UserEntity({required this.email}) {
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email)) {
      throw Exception('Invalid email');
    }
  }

  final String email;
}
```

**Por quê evitar?**

- Validações complexas devem ficar nos **UseCases** ou **Value Objects**
- Entities devem ser simples e focadas em carregar dados
- Facilita testes e manutenção

---

## � Exemplos de Uso

### Criando uma Entity

```dart
final user = UserEntity(
  id: '123',
  name: 'João Silva',
  email: 'joao@example.com',
  cpf: '123.456.789-00',
  phone: '(11) 98765-4321',
  birth: DateTime(1990, 5, 15),
  lastLoginAt: DateTime.now(),
  status: UserStatus.active,
  gender: UserGenderType.male,
  isActive: true,
  score: 95.5,
  address: AddressEntity(
    street: 'Rua das Flores',
    number: '123',
    city: 'São Paulo',
    state: 'SP',
    postalCode: '01234-567',
    country: 'Brasil',
    neighborhood: 'Centro',
    complement: 'Apto 45',
  ),
  tags: ['premium', 'verified'],
);
```

### Modificando com copyWith

```dart
// Atualizar apenas o status
final updatedUser = user.copyWith(
  status: UserStatus.inactive,
);

// Atualizar múltiplos campos
final modifiedUser = user.copyWith(
  name: 'João Pedro Silva',
  email: 'joaopedro@example.com',
  phone: '(11) 91234-5678',
  lastLoginAt: DateTime.now(),
);

// Remover campo opcional (setar como null)
final userWithoutPhone = user.copyWith(
  phone: null,
);
```

### Composição de Entities

```dart
// Entity que compõe outras entities
final session = SessionEntity(
  user: user,
  token: TokenEntity(
    accessToken: 'abc123...',
    refreshToken: 'xyz789...',
    expiresAt: DateTime.now().add(Duration(hours: 1)),
  ),
);
```

---

## 📋 Checklist de Implementação

Ao criar uma nova Entity, verifique:

- [ ] **Constructor `const`** implementado
- [ ] **Todos os campos são `final`**
- [ ] **Named parameters com `required`** para campos obrigatórios
- [ ] **Nullable (`?`)** para campos opcionais
- [ ] **Método `copyWith`** implementado corretamente
- [ ] **Imports apenas do domain** (enums, outras entities)
- [ ] **Nomenclatura segue padrão** (`{Nome}Entity`)
- [ ] **Arquivo nomeado corretamente** (`{nome}_entity.dart`)
- [ ] **Tipagem forte** (Enums ao invés de Strings quando apropriado)
- [ ] **Sem dependências externas** (frameworks, libs)
- [ ] **Sem lógica de serialização** (fica nos Models)

---

## 🚀 Benefícios das Entities Puras

### ✅ Testabilidade

```dart
// Fácil criar instâncias para testes
final testUser = UserEntity(
  id: 'test-123',
  name: 'Test User',
  email: 'test@example.com',
  cpf: '000.000.000-00',
  birth: DateTime(1990, 1, 1),
  status: UserStatus.active,
  gender: UserGenderType.male,
  isActive: true,
  score: 100.0,
  address: testAddress,
  tags: ['test'],
);

// Fácil verificar igualdade
expect(user.id, equals('test-123'));
expect(user.name, equals('Test User'));
```

### ✅ Imutabilidade

```dart
// Não é possível modificar acidentalmente
user.name = 'Outro Nome';  // ❌ Erro de compilação!

// Modificações são explícitas e criam novas instâncias
final newUser = user.copyWith(name: 'Outro Nome');  // ✅ Correto
```

### ✅ Reutilização

```dart
// AddressEntity pode ser usada em múltiplos contextos
class UserEntity {
  final AddressEntity address;
}

class CompanyEntity {
  final AddressEntity address;
}

class DistributorEntity {
  final AddressEntity address;
}
```

### ✅ Manutenibilidade

- Mudanças em uma Entity não afetam outras camadas
- Fácil adicionar novos campos sem quebrar código existente
- Código limpo e fácil de entender

---

## 🔗 Próximos Passos

Após criar suas Entities, você pode:

1. **[Definir Enums](./enums.md)** - Valores constantes usados nas Entities
2. **[Criar Failures](./failures.md)** - Tipos de erro específicos do domínio
3. **[Definir Repository Interfaces](./i_repositories.md)** - Contratos de acesso aos dados
4. **[Definir UseCase Interfaces](./i_usecases.md)** - Contratos de operações de negócio

---

_Entities são a base do seu domínio. Mantenha-as puras, simples e focadas em representar conceitos de negócio._
