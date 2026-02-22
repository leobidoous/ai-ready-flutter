# Models - Infrastructure Layer

## 📋 O que são Models?

**Models** são adaptadores que fazem a transformação entre **dados externos** (JSON da API) e **Entities** (objetos de negócio). Eles estendem Entities e adicionam capacidades de serialização/deserialização, mantendo a compatibilidade com o Domain.

### 🎯 Responsabilidades

**✅ O que Models FAZEM:**

- **Estendem Entities** - Herdam todos os campos e comportamentos
- **Serializam** - Transformam Entity → JSON (toMap, toCreate, toUpdate)
- **Deserializam** - Transformam JSON → Entity (fromMap)
- **Tratam dados nulos** - Fornecem valores padrão seguros
- **Formatam dados** - Limpam CPF, telefone, datas, etc.
- **Implementam Equatable** - Comparação de igualdade

**❌ O que Models NÃO FAZEM:**

- Não contêm regras de negócio (ficam nas Entities ou UseCases)
- Não fazem comunicação externa (isso é DataSource)
- Não conhecem detalhes de HTTP ou APIs

---

## 🏗️ Exemplo Completo: UserModel

Este exemplo demonstra **todos os elementos** de um Model bem estruturado, correspondendo ao UserEntity documentado:

```dart
import 'package:base_core/base_core.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/enums/user_gender_type.dart';
import '../../domain/enums/user_status.dart';
import 'address_model.dart';

class UserModel extends UserEntity with EquatableMixin {
  // 1. Constructor - Passa todos os parâmetros para a Entity
  UserModel({
    required super.id,
    required super.name,
    required super.email,
    required super.cpf,
    required super.birth,
    required super.status,
    required super.gender,
    required super.isActive,
    required super.score,
    required super.address,
    required super.tags,
    super.phone,
    super.lastLoginAt,
  });

  // 2. fromMap - Deserialização (JSON → Model)
  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      // Strings com fallback
      id: map['id']?.toString() ?? '',
      name: map['name']?.toString() ?? '',
      email: map['email']?.toString() ?? '',
      cpf: map['cpf']?.toString() ?? '',

      // String? nullable
      phone: map['phone']?.toString(),

      // DateTime com tratamento
      birth: DateFormat.tryParseOrDateNow(map['birthDate']?.toString()),

      // DateTime? nullable
      lastLoginAt: map['lastLoginAt'] != null
          ? DateTime.tryParse(map['lastLoginAt'].toString())
          : null,

      // Enums com fromJson
      status: UserStatus.fromJson(map['status']?.toString()),
      gender: UserGenderType.fromJson(map['gender']?.toString()),

      // bool
      isActive: map['isActive'] == true,

      // double/int
      score: (map['score'] ?? 0).toDouble(),

      // Entity aninhada
      address: AddressModel.fromMap(
        map['address'] is Map ? map['address'] : {},
      ),

      // List
      tags: map['tags'] is List
          ? List<String>.from(map['tags'])
          : [],
    );
  }

  // 3. fromEntity - Conversão (Entity → Model)
  factory UserModel.fromEntity(UserEntity entity) {
    return UserModel(
      id: entity.id,
      name: entity.name,
      email: entity.email,
      cpf: entity.cpf,
      phone: entity.phone,
      birth: entity.birth,
      lastLoginAt: entity.lastLoginAt,
      status: entity.status,
      gender: entity.gender,
      isActive: entity.isActive,
      score: entity.score,
      address: entity.address,
      tags: entity.tags,
    );
  }

  // 4. toEntity - Conversão (Model → Entity)
  UserEntity get toEntity => this;

  // 5. toMap - Serialização completa (Model → JSON)
  Map<String, dynamic> get toMap {
    return {
      'id': id,
      'name': name,
      'email': email,
      'cpf': cpf.replaceAll(RegExp('[^0-9]'), ''),  // Remove formatação
      'phone': phone,
      'birthDate': birth.toIso8601String(),
      'lastLoginAt': lastLoginAt?.toIso8601String(),
      'status': status.toJson,
      'gender': gender.toJson,
      'isActive': isActive,
      'score': score,
      'address': AddressModel.fromEntity(address).toMap,
      'tags': tags,
    };
  }

  // 6. toCreate - Serialização para criação (POST)
  Map<String, dynamic> get toCreate {
    return {
      'name': name,
      'email': email,
      'cpf': cpf.trim().replaceAll(RegExp('[^0-9]'), ''),
      'phone': phone?.trim().replaceAll(RegExp('[^0-9]'), ''),
      'birthDate': birth.toIso8601String(),
      'gender': gender.toJson,
      'address': AddressModel.fromEntity(address).toMap,
    };
  }

  // 7. toUpdate - Serialização para atualização (PUT/PATCH)
  Map<String, dynamic> get toUpdate {
    return {
      if (name.isNotEmpty) 'name': name,
      if (email.isNotEmpty) 'email': email,
      if (phone != null && phone!.isNotEmpty)
        'phone': phone!.trim().replaceAll(RegExp('[^0-9]'), ''),
      'birthDate': birth.toIso8601String(),
      'gender': gender.toJson,
      'address': AddressModel.fromEntity(address).toMap,
    };
  }

  // 8. Equatable - Comparação de igualdade
  @override
  List<Object?> get props => [
        id,
        name,
        email,
        cpf,
        phone,
        birth,
        lastLoginAt,
        status,
        gender,
        isActive,
        score,
        address,
        tags,
      ];

  @override
  bool? get stringify => true;
}
```

### 🔑 Elementos Essenciais

1. **Extends Entity** - Herda todos os campos da Entity
2. **Mixin EquatableMixin** - Comparação de igualdade
3. **fromMap** - Deserialização JSON → Model (com tratamento de nulos)
4. **fromEntity** - Conversão Entity → Model
5. **toEntity** - Conversão Model → Entity (geralmente `this`)
6. **toMap** - Serialização completa Model → JSON
7. **toCreate/toUpdate** - Serializações específicas para operações
8. **props** - Lista de propriedades para Equatable

---

## 🎯 Tratamento de Dados por Tipo

### 1. **String** - Com fallback

```dart
id: map['id']?.toString() ?? '',
name: map['name']?.toString() ?? '',
```

### 2. **String?** - Nullable

```dart
phone: map['phone']?.toString(),
```

### 3. **DateTime** - Com tratamento

```dart
birth: DateFormat.tryParseOrDateNow(map['birthDate']?.toString()),
```

### 4. **DateTime?** - Nullable

```dart
lastLoginAt: map['lastLoginAt'] != null
    ? DateTime.tryParse(map['lastLoginAt'].toString())
    : null,
```

### 5. **Enum** - Com fromJson

```dart
status: UserStatus.fromJson(map['status']?.toString()),
gender: UserGenderType.fromJson(map['gender']?.toString()),
```

### 6. **bool** - Com verificação

```dart
isActive: map['isActive'] == true,
```

### 7. **int/double** - Com fallback

```dart
score: (map['score'] ?? 0).toDouble(),
age: map['age'] as int? ?? 0,
```

### 8. **Entity** - Model aninhado

```dart
address: AddressModel.fromMap(
  map['address'] is Map ? map['address'] : {},
),
```

### 9. **List** - Com verificação

```dart
tags: map['tags'] is List
    ? List<String>.from(map['tags'])
    : [],
```

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo        | Padrão              | Exemplo                                 |
| ----------- | ------------------- | --------------------------------------- |
| **Classe**  | `{Nome}Model`       | `UserModel`, `AddressModel`             |
| **Arquivo** | `{nome}_model.dart` | `user_model.dart`, `address_model.dart` |

### ✅ Estrutura do Arquivo

```dart
// 1. Imports
import 'package:base_core/base_core.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/enums/user_status.dart';

// 2. Classe extends Entity with EquatableMixin
class UserModel extends UserEntity with EquatableMixin {
  // 3. Constructor
  UserModel({required super.id, required super.name, ...});

  // 4. fromMap
  factory UserModel.fromMap(Map<String, dynamic> map) { ... }

  // 5. fromEntity
  factory UserModel.fromEntity(UserEntity entity) { ... }

  // 6. toEntity
  UserEntity get toEntity => this;

  // 7. toMap
  Map<String, dynamic> get toMap { ... }

  // 8. toCreate (opcional)
  Map<String, dynamic> get toCreate { ... }

  // 9. toUpdate (opcional)
  Map<String, dynamic> get toUpdate { ... }

  // 10. Equatable props
  @override
  List<Object?> get props => [id, name, ...];

  @override
  bool? get stringify => true;
}
```

---

## 📋 Checklist de Implementação

Ao criar um Model:

- [ ] **Extends Entity** correspondente
- [ ] **Mixin EquatableMixin** adicionado
- [ ] **Constructor** passa todos os parâmetros para `super`
- [ ] **fromMap** implementado com tratamento de nulos
- [ ] **fromEntity** implementado
- [ ] **toEntity** implementado (geralmente `this`)
- [ ] **toMap** implementado
- [ ] **toCreate/toUpdate** implementados (se necessário)
- [ ] **props** lista todas as propriedades
- [ ] **stringify** definido como `true`
- [ ] **Tratamento de nulos** em todos os campos
- [ ] **Formatação de dados** (CPF, telefone, datas)

---

## 🚀 Benefícios dos Models

### ✅ Separação de Responsabilidades

- **Entity**: Regras de negócio puras
- **Model**: Serialização/deserialização

### ✅ Tratamento Robusto de Dados

```dart
// Trata nulos, tipos incorretos, dados faltando
factory UserModel.fromMap(Map<String, dynamic> map) {
  return UserModel(
    id: map['id']?.toString() ?? '',  // Nunca null
    name: map['name']?.toString() ?? '',  // Sempre String
    address: AddressModel.fromMap(
      map['address'] is Map ? map['address'] : {},  // Sempre Map
    ),
  );
}
```

### ✅ Flexibilidade de Serialização

```dart
// Diferentes formatos para diferentes operações
final createPayload = model.toCreate;  // Campos para criar
final updatePayload = model.toUpdate;  // Campos para atualizar
final fullData = model.toMap;  // Todos os campos
```

### ✅ Comparação de Igualdade

```dart
// Equatable permite comparação fácil
final user1 = UserModel(id: '1', name: 'João', ...);
final user2 = UserModel(id: '1', name: 'João', ...);

print(user1 == user2);  // true (mesmos valores)
```

---

## � Próximos Passos

1. **[Implementar Repository](./repositories.md)** - Usar Models para transformar dados
2. **[Implementar UseCase](./usecases.md)** - Orquestrar Repository
3. **[Implementar DataSource](../data/datasources.md)** - Comunicação real

---

_Models são adaptadores essenciais que fazem a ponte entre dados externos (JSON) e objetos de negócio (Entities), mantendo o Domain puro e independente._
