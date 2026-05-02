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
- Não montam maps inline para sub-entities — sempre delegam para o sub-model correspondente (usar `SubModel.fromEntity(e).toMap`)

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

  // 2. fromMap - Deserialização (Map → Model)
  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      // Strings com fallback e ?.toString()
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

      // double com tryParse
      score: double.tryParse(map['score']?.toString() ?? '0.0') ?? 0.0,

      // Entity aninhada com verificação de tipo
      address: AddressModel.fromMap(
        map['address'] is Map ? map['address'] : {},
      ),

      // List com verificação de tipo
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

  // 5. toMap - Serialização completa (Model → Map)
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

Todos os Models DEVEM ter os seguintes elementos. Nenhum é opcional:

1. **Extends Entity** - Herda todos os campos da Entity
2. **Mixin EquatableMixin** - Comparação de igualdade (com `props` e `stringify`)
3. **fromMap** - Deserialização JSON → Model (com tratamento de nulos)
4. **fromEntity** - Conversão Entity → Model
5. **toEntity** - Conversão Model → Entity (geralmente `this`)
6. **toMap** - Serialização completa Model → JSON
7. **toCreate/toUpdate** - Serializações específicas para operações (quando aplicável)
8. **props** - Lista de propriedades para Equatable

---

## 🎯 Tratamento de Dados por Tipo

### 1. **String** - Com fallback

```dart
id: map['id']?.toString() ?? '',
name: map['name']?.toString() ?? '',
```

**Importante**: Sempre use `?.toString()` antes do fallback para evitar erros de cast quando a API retorna tipos inesperados.

### 2. **String?** - Nullable

```dart
phone: map['phone']?.toString(),
```

### 3. **int** - Com tryParse e fallback

```dart
age: int.tryParse(map['age']?.toString() ?? '0') ?? 0,
count: int.tryParse(map['count']?.toString() ?? '0') ?? 0,
```

**Importante**: Use `int.tryParse` ao invés de `as int` para evitar erros quando a API retorna strings ou outros tipos.

### 4. **double** - Com tryParse e fallback

```dart
score: double.tryParse(map['score']?.toString() ?? '0.0') ?? 0.0,
price: double.tryParse(map['price']?.toString() ?? '0.0') ?? 0.0,
```

### 5. **DateTime** - Com tratamento

```dart
birth: DateFormat.tryParseOrDateNow(map['birthDate']?.toString()),
```

### 6. **DateTime?** - Nullable

```dart
lastLoginAt: map['lastLoginAt'] != null
    ? DateTime.tryParse(map['lastLoginAt'].toString())
    : null,
```

### 7. **Enum** - Com fromJson

```dart
status: UserStatus.fromJson(map['status']?.toString()),
gender: UserGenderType.fromJson(map['gender']?.toString()),
```

### 8. **bool** - Com verificação

```dart
isActive: map['isActive'] == true,
```

### 9. **Entity** - Model aninhado

```dart
address: AddressModel.fromMap(
  map['address'] is Map ? map['address'] : {},
),
```

### 10. **List** - Com verificação

```dart
tags: map['tags'] is List
    ? List<String>.from(map['tags'])
    : [],
```

### 11. **List<Entity>** - Com validação de tipo e tratamento de erros

```dart
// Função helper para parsing seguro
List<ItemModel> parseItems() {
  final itemsData = map['items'];
  if (itemsData is! List) return [];

  try {
    return itemsData
        .where((e) => e is Map<String, dynamic>)
        .map((e) => ItemModel.fromMap(e as Map<String, dynamic>))
        .toList();
  } catch (e) {
    return [];
  }
}

items: parseItems(),
```

### 📋 Exemplo Completo: PaginatedResponseModel

Este exemplo demonstra todas as validações robustas em ação:

```dart
factory PaginatedResponseModel.fromMap(
  Map<String, dynamic> map,
  T Function(Map<String, dynamic>) fromMap,
) {
  // Safely parse rows - validate it's a List first
  List<T> parseRows() {
    final rowsData = map['rows'];
    if (rowsData is! List) return [];

    try {
      return rowsData
          .where((e) => e is Map<String, dynamic>)
          .map((e) => fromMap(e as Map<String, dynamic>))
          .toList();
    } catch (e) {
      return [];
    }
  }

  return PaginatedResponseModel<T>(
    // Lista com validação completa de tipo
    rows: parseRows(),

    // Inteiros com tryParse e fallback
    offset: int.tryParse(map['offset']?.toString() ?? '0') ?? 0,
    limit: int.tryParse(map['limit']?.toString() ?? '10') ?? 10,
    count: int.tryParse(map['count']?.toString() ?? '0') ?? 0,
  );
}
```

**Casos tratados por este código**:

- ✅ `rows` como string: `"invalid"` → `[]`
- ✅ `rows` como null: `null` → `[]`
- ✅ `rows` como número: `123` → `[]`
- ✅ `rows` como objeto: `{}` → `[]`
- ✅ `rows` com itens inválidos: `[1, "text", {...}]` → filtra apenas Maps válidos
- ✅ Erro no `fromMap` de um item: captura e retorna lista parcial
- ✅ `offset/limit/count` como string: `"10"` → `10`
- ✅ `offset/limit/count` como null: `null` → valores padrão (0, 10, 0)
- ✅ `offset/limit/count` como double: `10.5` → `10`

---

## 🎯 Valores Padrão Recomendados

### Por que validar?

APIs podem retornar dados em formatos inesperados:

- Números como strings: `"123"` ao invés de `123`
- Strings como números: `123` ao invés de `"123"`
- Valores nulos inesperados
- Tipos completamente diferentes

### ✅ Padrões de Validação Recomendados

#### 1. **Strings** - Sempre use `?.toString()`

```dart
// ❌ ERRADO - Pode quebrar se vier número
id: map['id'] as String,

// ✅ CORRETO - Converte qualquer tipo para string
id: map['id']?.toString() ?? '',
```

#### 2. **Inteiros** - Sempre use `int.tryParse`

```dart
// ❌ ERRADO - Quebra se vier string "123"
offset: map['offset'] as int,

// ✅ CORRETO - Converte string ou número para int
offset: int.tryParse(map['offset']?.toString() ?? '0') ?? 0,
```

#### 3. **Doubles** - Sempre use `double.tryParse`

```dart
// ❌ ERRADO - Quebra se vier string "12.5"
price: map['price'] as double,

// ✅ CORRETO - Converte string ou número para double
price: double.tryParse(map['price']?.toString() ?? '0.0') ?? 0.0,
```

#### 4. **Listas** - Sempre verifique o tipo primeiro

```dart
// ❌ ERRADO - Quebra se vier string, null ou não for lista
rows: (map['rows'] as List<dynamic>)
    .map((e) => ItemModel.fromMap(e))
    .toList(),

// ✅ CORRETO - Verifica tipo antes de processar
List<ItemModel> parseRows() {
  final rowsData = map['rows'];
  if (rowsData is! List) return [];

  try {
    return rowsData
        .where((e) => e is Map<String, dynamic>)
        .map((e) => ItemModel.fromMap(e as Map<String, dynamic>))
        .toList();
  } catch (e) {
    return [];
  }
}

rows: parseRows(),
```

**Por que essa abordagem?**

- Valida se `rows` é realmente uma `List` (pode vir como string, null, número, etc.)
- Filtra apenas itens que são `Map<String, dynamic>` (ignora itens inválidos)
- Usa try-catch para capturar erros no `fromMap` de itens individuais
- Retorna lista vazia em qualquer erro ao invés de quebrar a aplicação

#### 5. **Maps aninhados** - Sempre verifique o tipo

```dart
// ❌ ERRADO - Quebra se vier null ou não for Map
address: AddressModel.fromMap(map['address']),

// ✅ CORRETO - Verifica tipo e fornece Map vazio se null
address: AddressModel.fromMap(
  map['address'] is Map ? map['address'] : {},
),
```

### 📋 Exemplo Completo: PaginatedResponseModel

```dart
factory PaginatedResponseModel.fromMap(
  Map<String, dynamic> map,
  T Function(Map<String, dynamic>) fromMap,
) {
  return PaginatedResponseModel<T>(
    // Lista com verificação de tipo e fallback
    rows: (map['rows'] as List<dynamic>?)
            ?.map((e) => fromMap(e as Map<String, dynamic>))
            .toList() ??
        [],

    // Inteiros com tryParse e fallback
    offset: int.tryParse(map['offset']?.toString() ?? '0') ?? 0,
    limit: int.tryParse(map['limit']?.toString() ?? '10') ?? 10,
    count: int.tryParse(map['count']?.toString() ?? '0') ?? 0,
  );
}
```

### 🎯 Valores Padrão Recomendados

| Tipo     | Valor Padrão        | Exemplo                                                     |
| -------- | ------------------- | ----------------------------------------------------------- |
| String   | `''` (string vazia) | `map['name']?.toString() ?? ''`                             |
| int      | `0`                 | `int.tryParse(map['age']?.toString() ?? '0') ?? 0`          |
| double   | `0.0`               | `double.tryParse(map['price']?.toString() ?? '0.0') ?? 0.0` |
| bool     | `false`             | `map['isActive'] == true`                                   |
| List     | `[]` (lista vazia)  | `(map['items'] as List?) ?? []`                             |
| Map      | `{}` (map vazio)    | `map['data'] is Map ? map['data'] : {}`                     |
| DateTime | `DateTime.now()`    | `DateFormat.tryParseOrDateNow(map['date']?.toString())`     |

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo               | Padrão              | Exemplo                                           |
| ------------------ | ------------------- | ------------------------------------------------- |
| **Classe**         | `{Nome}Model`       | `UserModel`, `AddressModel`                       |
| **Arquivo**        | `{nome}_model.dart` | `user_model.dart`, `address_model.dart`           |
| **Deserialização** | `fromMap`           | `factory Model.fromMap(Map<String, dynamic> map)` |
| **Serialização**   | `toMap`             | `Map<String, dynamic> get toMap`                  |

**Importante**: Use `toMap/fromMap` ao invés de `toJson/fromJson` para deixar claro que estamos trabalhando com `Map<String, dynamic>`, não com strings JSON. A conversão para JSON string é feita pela camada de HTTP (DataSource).

### ✅ Diferença entre Map e JSON

```dart
// Map<String, dynamic> - Estrutura de dados Dart
final map = {'name': 'João', 'age': 30};

// JSON String - Texto serializado
final jsonString = '{"name":"João","age":30}';

// Model trabalha com Map, não com JSON String
factory UserModel.fromMap(Map<String, dynamic> map) { ... }
Map<String, dynamic> get toMap { ... }

// DataSource faz a conversão Map ↔ JSON String
final response = await http.post(
  url,
  body: jsonEncode(model.toMap),  // Map → JSON String
);
final model = UserModel.fromMap(
  jsonDecode(response.body),  // JSON String → Map
);
```

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

### ✅ Organização Estética de Parâmetros

**IMPORTANTE**: Para manter consistência visual e facilitar leitura, organize os parâmetros do menor para o maior número de caracteres na linha.

```dart
// ✅ CORRETO - Organizado por tamanho de linha (menor → maior)
class UserModel extends UserEntity with EquatableMixin {
  UserModel({
    super.phone,              // 16 chars - opcional primeiro
    required super.id,        // 21 chars
    required super.name,      // 23 chars
    required super.email,     // 24 chars
    required super.birth,     // 24 chars
    required super.status,    // 25 chars
    required super.gender,    // 25 chars
    required super.address,   // 26 chars
    required super.isActive,  // 27 chars
  });

  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      id: map['id']?.toString() ?? '',
      name: map['name']?.toString() ?? '',
      email: map['email']?.toString() ?? '',
      phone: map['phone']?.toString(),
      birth: DateFormat.tryParseOrDateNow(map['birthDate']?.toString()),
      status: UserStatus.fromJson(map['status']?.toString()),
      gender: UserGenderType.fromJson(map['gender']?.toString()),
      isActive: map['isActive'] == true,
      address: AddressModel.fromMap(
        map['address'] is Map ? map['address'] : {},
      ),
    );
  }

  factory UserModel.fromEntity(UserEntity entity) {
    return UserModel(
      id: entity.id,
      name: entity.name,
      email: entity.email,
      phone: entity.phone,
      birth: entity.birth,
      status: entity.status,
      gender: entity.gender,
      address: entity.address,
      isActive: entity.isActive,
    );
  }

  Map<String, dynamic> get toMap {
    return {
      'id': id,
      'name': name,
      'email': email,
      'phone': phone,
      'isActive': isActive,
      'birthDate': birth.toIso8601String(),
      'status': status.toJson,
      'gender': gender.toJson,
      'cpf': cpf.replaceAll(RegExp('[^0-9]'), ''),
      'address': AddressModel.fromEntity(address).toMap,
    };
  }
}

// ❌ EVITAR - Sem organização visual
class UserModel extends UserEntity with EquatableMixin {
  UserModel({
    required super.address,
    required super.id,
    required super.isActive,
    super.phone,
    required super.name,
    required super.email,
  });
}
```

**Regras de Organização:**

1. **Parâmetros opcionais primeiro** (sem `required`)
2. **Depois parâmetros obrigatórios** ordenados por tamanho de linha
3. **Contar caracteres da linha completa** incluindo `required`, nome do parâmetro e tipo
4. **Aplicar em**: constructors, `fromMap`, `fromEntity`, `toMap`, `toCreate`, `toUpdate`
5. **Em Maps**: organizar chaves por tamanho quando possível

**Benefícios:**

- ✅ Código visualmente mais limpo e organizado
- ✅ Facilita identificar parâmetros rapidamente
- ✅ Padrão consistente em toda a codebase
- ✅ Melhora legibilidade em code reviews

---

## 📋 Checklist de Implementação

Ao criar um Model, TODOS os itens abaixo são obrigatórios:

- [ ] **Extends Entity** correspondente
- [ ] **Mixin EquatableMixin** adicionado
- [ ] **Constructor** passa todos os parâmetros para `super`
- [ ] **fromMap** implementado com tratamento de nulos
- [ ] **fromEntity** implementado (converte Entity → Model)
- [ ] **toEntity** implementado (geralmente `get toEntity => this`)
- [ ] **toMap** implementado (serializa Model → Map)
- [ ] **toCreate/toUpdate** implementados (se necessário para operações específicas)
- [ ] **props** lista todas as propriedades
- [ ] **stringify** definido como `true`
- [ ] **Tratamento de nulos** em todos os campos do fromMap
- [ ] **Formatação de dados** (CPF, telefone, datas) no toMap
- [ ] **Sub-models delegam serialização** (usar `SubModel.fromEntity(e).toMap` ao invés de montar maps inline)

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
