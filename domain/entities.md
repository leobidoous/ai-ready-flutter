# 📦 Domain Layer - Entities

## 🎯 O que são Entities?

**Entities** são objetos de negócio puros que representam conceitos fundamentais do domínio. Elas carregam dados e regras de negócio básicas, sem dependências externas.

### Responsabilidades
- ✅ Representar objetos de negócio do domínio
- ✅ Carregar dados estruturados e tipados
- ✅ Garantir imutabilidade através de `final` fields
- ✅ Fornecer método `copyWith` para atualizações imutáveis
- ✅ Validações básicas com `assert` (quando necessário)

### Não Responsabilidades
- ❌ Lógica de persistência (isso é do Repository)
- ❌ Serialização JSON (isso é do Model na camada Infra)
- ❌ Comunicação com APIs (isso é do DataSource)
- ❌ Regras de negócio complexas (isso é do UseCase)

---

## 📋 Padrão de Nomenclatura

### Convenção de Nomes
```
[NomeDoConceito]Entity
```

### Exemplos
- `UserEntity` → arquivo: `user_entity.dart`
- `ProductEntity` → arquivo: `product_entity.dart`
- `AddressEntity` → arquivo: `address_entity.dart`
- `DistributorEntity` → arquivo: `distributor_entity.dart`
- `BankDataEntity` → arquivo: `bank_data_entity.dart`

---

## 🏗️ Estrutura Padrão de uma Entity

### Template Base
```dart
class [Nome]Entity {
  const [Nome]Entity({
    required this.campo1,
    required this.campo2,
    // ... outros campos
  });

  final TipoCampo1 campo1;
  final TipoCampo2 campo2;
  // ... outros campos

  [Nome]Entity copyWith({
    TipoCampo1? campo1,
    TipoCampo2? campo2,
    // ... outros campos
  }) {
    return [Nome]Entity(
      campo1: campo1 ?? this.campo1,
      campo2: campo2 ?? this.campo2,
      // ... outros campos
    );
  }
}
```

---

## 📝 Como Criar Entities a partir de JSON

### Passo 1: Analisar o JSON
Dado um JSON de resposta da API:

```json
{
  "id": "123",
  "name": "João Silva",
  "email": "joao@example.com",
  "phoneNumber": "+5511999999999",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Passo 2: Identificar os Campos e Tipos
| Campo JSON | Tipo Dart | Campo Entity |
|------------|-----------|--------------|
| `id` | `String` | `id` |
| `name` | `String` | `name` |
| `email` | `String` | `email` |
| `phoneNumber` | `String` | `phoneNumber` |
| `createdAt` | `DateTime` | `createdAt` |

### Passo 3: Criar a Entity
```dart
class UserEntity {
  const UserEntity({
    required this.id,
    required this.name,
    required this.email,
    required this.phoneNumber,
    required this.createdAt,
  });

  final String id;
  final String name;
  final String email;
  final String phoneNumber;
  final DateTime createdAt;

  UserEntity copyWith({
    String? id,
    String? name,
    String? email,
    String? phoneNumber,
    DateTime? createdAt,
  }) {
    return UserEntity(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      phoneNumber: phoneNumber ?? this.phoneNumber,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}
```

---

## 🔗 Entities Compostas (Nested Objects)

### JSON com Objetos Aninhados
```json
{
  "id": "123",
  "name": "João Silva",
  "address": {
    "street": "Rua A",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01234-567"
  }
}
```

### Criar Entity Separada para Objeto Aninhado
```dart
// address_entity.dart
class AddressEntity {
  const AddressEntity({
    required this.street,
    required this.city,
    required this.state,
    required this.zipCode,
  });

  final String street;
  final String city;
  final String state;
  final String zipCode;

  AddressEntity copyWith({
    String? street,
    String? city,
    String? state,
    String? zipCode,
  }) {
    return AddressEntity(
      street: street ?? this.street,
      city: city ?? this.city,
      state: state ?? this.state,
      zipCode: zipCode ?? this.zipCode,
    );
  }
}
```

### Entity Principal Referenciando a Entity Aninhada
```dart
// user_entity.dart
import 'address_entity.dart';

class UserEntity {
  const UserEntity({
    required this.id,
    required this.name,
    required this.address,
  });

  final String id;
  final String name;
  final AddressEntity address;

  UserEntity copyWith({
    String? id,
    String? name,
    AddressEntity? address,
  }) {
    return UserEntity(
      id: id ?? this.id,
      name: name ?? this.name,
      address: address ?? this.address,
    );
  }
}
```

---

## 📚 Exemplos Práticos do Projeto

### Exemplo 1: Entity Simples - AddressEntity
```dart
class AddressEntity {
  AddressEntity({
    required this.city,
    required this.state,
    required this.street,
    required this.number,
    required this.country,
    required this.complement,
    required this.postalCode,
    required this.neighborhood,
  });

  final String city;
  final String state;
  final String street;
  final String number;
  final String country;
  final String complement;
  final String postalCode;
  final String neighborhood;

  AddressEntity copyWith({
    String? city,
    String? state,
    String? street,
    String? number,
    String? country,
    String? complement,
    String? postalCode,
    String? neighborhood,
  }) {
    return AddressEntity(
      city: city ?? this.city,
      state: state ?? this.state,
      street: street ?? this.street,
      number: number ?? this.number,
      country: country ?? this.country,
      complement: complement ?? this.complement,
      postalCode: postalCode ?? this.postalCode,
      neighborhood: neighborhood ?? this.neighborhood,
    );
  }
}
```

### Exemplo 2: Entity Composta - UserEntity
```dart
import '../enums/user_gender_type.dart';
import 'address_entity.dart';
import 'user_company_entity.dart';
import 'user_role_entity.dart';

class UserEntity {
  const UserEntity({
    required this.id,
    required this.cpf,
    required this.name,
    required this.email,
    required this.phone,
    required this.birth,
    required this.gender,
    required this.address,
    required this.role,
    required this.company,
  });

  final String id;
  final String cpf;
  final String name;
  final String email;
  final String phone;
  final DateTime birth;
  final UserGenderType gender;
  final AddressEntity address;
  final UserRoleEntity role;
  final UserCompanyEntity company;

  UserEntity copyWith({
    String? id,
    String? cpf,
    String? name,
    String? email,
    String? phone,
    DateTime? birth,
    UserGenderType? gender,
    AddressEntity? address,
    UserRoleEntity? role,
    UserCompanyEntity? company,
  }) {
    return UserEntity(
      id: id ?? this.id,
      cpf: cpf ?? this.cpf,
      name: name ?? this.name,
      email: email ?? this.email,
      phone: phone ?? this.phone,
      birth: birth ?? this.birth,
      gender: gender ?? this.gender,
      address: address ?? this.address,
      role: role ?? this.role,
      company: company ?? this.company,
    );
  }
}
```

### Exemplo 3: Entity com Múltiplos Objetos Aninhados - DistributorEntity
```dart
import 'address_entity.dart';
import 'bank_data_entity.dart';

class DistributorEntity {
  DistributorEntity({
    required this.id,
    required this.cnpj,
    required this.name,
    required this.email,
    required this.phoneNumber,
    required this.uniqueCode,
    required this.address,
    required this.bankData,
    required this.stage,
    required this.status,
    required this.dateStatus,
  });

  final String id;
  final String cnpj;
  final String name;
  final String email;
  final String phoneNumber;
  final String uniqueCode;
  final AddressEntity address;
  final BankDataEntity bankData;
  final String stage;
  final String status;
  final DateTime dateStatus;

  DistributorEntity copyWith({
    String? id,
    String? cnpj,
    String? name,
    String? email,
    String? phoneNumber,
    String? uniqueCode,
    AddressEntity? address,
    BankDataEntity? bankData,
    String? stage,
    String? status,
    DateTime? dateStatus,
  }) {
    return DistributorEntity(
      id: id ?? this.id,
      cnpj: cnpj ?? this.cnpj,
      name: name ?? this.name,
      email: email ?? this.email,
      phoneNumber: phoneNumber ?? this.phoneNumber,
      uniqueCode: uniqueCode ?? this.uniqueCode,
      address: address ?? this.address,
      bankData: bankData ?? this.bankData,
      stage: stage ?? this.stage,
      status: status ?? this.status,
      dateStatus: dateStatus ?? this.dateStatus,
    );
  }
}
```

---

## ✅ Checklist para Criar uma Entity

- [ ] Nome segue padrão `[Nome]Entity`
- [ ] Arquivo segue padrão `[nome]_entity.dart`
- [ ] Todos os campos são `final`
- [ ] Constructor usa `required` para campos obrigatórios
- [ ] Constructor é `const` quando possível
- [ ] Método `copyWith` implementado para todos os campos
- [ ] Imports apenas de outras entities, enums ou value objects do domain
- [ ] Sem lógica de serialização (JSON)
- [ ] Sem dependências externas (packages)
- [ ] Tipos primitivos ou outras entities do domain

---

## 🚫 Anti-Patterns - O que NÃO fazer

### ❌ Não adicionar serialização JSON
```dart
// ❌ ERRADO - Entity não deve ter fromJson/toJson
class UserEntity {
  factory UserEntity.fromJson(Map<String, dynamic> json) { ... }
  Map<String, dynamic> toJson() { ... }
}
```

### ❌ Não adicionar lógica de negócio complexa
```dart
// ❌ ERRADO - Lógica complexa deve estar no UseCase
class UserEntity {
  bool canPurchase() {
    // lógica complexa aqui
  }
}
```

### ❌ Não usar campos mutáveis
```dart
// ❌ ERRADO - Campos devem ser final
class UserEntity {
  String name; // sem final
}
```

### ❌ Não importar packages externos
```dart
// ❌ ERRADO - Entity não deve depender de packages externos
import 'package:dio/dio.dart';
import '../infra/models/user_model.dart';
```

---

## 🎯 Regras de Ouro

1. **Imutabilidade**: Todos os campos são `final`
2. **Pureza**: Sem dependências externas, apenas domain
3. **Simplicidade**: Apenas dados e copyWith
4. **Tipagem Forte**: Use tipos específicos (DateTime, Enums, outras Entities)
5. **Nomenclatura**: Sempre `[Nome]Entity` e `[nome]_entity.dart`

---

## 📖 Próximos Passos

Após criar a Entity no Domain:
1. Criar o Model correspondente na camada Infra (com fromJson/toJson)
2. Criar Failures específicas se necessário
3. Criar Repository Interface que usa a Entity
4. Criar UseCase Interface que usa a Entity
