# Infrastructure Models - Clean Architecture

## 📋 Visão Geral

Os **Models** são implementações concretas das entities que lidam com a serialização/deserialização de dados e adaptação entre camadas. Eles servem como ponte entre o mundo externo (APIs, banco de dados) e as entities do domínio.

### 🎯 Propósito

- **Serialização**: Converter entities para/de formatos externos (JSON, XML, etc.)
- **Adaptação**: Adaptar dados externos para o formato esperado pelas entities
- **Validação de Dados**: Tratar dados inconsistentes vindos de fontes externas
- **Mapeamento**: Mapear campos com nomes diferentes entre entity e fonte externa

### 📍 Localização na Arquitetura

```
lib/
└── src/
    └── infra/
        └── models/
            ├── user_model.dart
            ├── product_model.dart
            └── order_model.dart
```

### 📋 Regras de Nomenclatura

**Classes:**
- ✅ **Sufixo obrigatório**: Sempre terminar com `Model` (singular)
- ✅ **PascalCase**: `UserModel`, `ProductModel`, `PaymentPlanModel`
- ❌ **Plurais**: Nunca usar `Models` (plural)

**Arquivos:**
- ✅ **snake_case**: Converter o nome da classe para snake_case
- ✅ **Nome da classe principal**: O arquivo deve ter o nome da classe principal
- ✅ **Exemplos corretos**: 
  - `UserModel` → `user_model.dart`
  - `PaymentPlanModel` → `payment_plan_model.dart`
  - `SimulationFiltersResponseModel` → `simulation_filters_response_model.dart`
- ❌ **Exemplos incorretos**:
  - `user_models.dart` (plural)
  - `payment_models.dart` (genérico)
  - `userModel.dart` (camelCase)

**Organização com part/part of:**
- ✅ **Quando usar**: Para models complexos com muitas classes relacionadas
- ✅ **Arquivo principal**: Contém o model principal e os `part` imports
- ✅ **Arquivos part**: Cada model complementar em arquivo separado com `part of`
- ✅ **Estrutura exemplo**:
  ```
  simulation_filters_response_model.dart  // Principal
  ├── eligible_scholarship_model.dart     // part of
  ├── awarded_scholarship_model.dart      // part of
  ├── simulation_success_model.dart       // part of
  └── payment_plan_model.dart            // part of
  ```

---

## 🏗️ Estrutura Base de um Model

### Regras Fundamentais

1. **Herança**: Models estendem suas respectivas Entities
2. **Equatable**: Implementar comparação via EquatableMixin
3. **Factory Constructors**: Para criação a partir de Map e Entity
4. **Serialização**: Métodos toMap/toJson para conversão
5. **Tratamento de Nulos**: Valores padrão para campos opcionais

### Template Base

```dart
import 'package:base_core/base_core.dart';
import '../../domain/entities/[nome]_entity.dart';

class [Nome]Model extends [Nome]Entity with EquatableMixin {
  const [Nome]Model({
    required super.propriedade1,
    required super.propriedade2,
    super.propriedadeOpcional,
  });

  /// Factory para criar model a partir de Map (JSON/API)
  factory [Nome]Model.fromMap(Map<String, dynamic> map) {
    return [Nome]Model(
      propriedade1: map['propriedade1'] ?? '',
      propriedade2: TipoComplexo.fromJson(map['propriedade2']),
      propriedadeOpcional: map['propriedade_opcional'],
    );
  }

  /// Factory para criar model a partir de Entity
  factory [Nome]Model.fromEntity([Nome]Entity entity) {
    return [Nome]Model(
      propriedade1: entity.propriedade1,
      propriedade2: entity.propriedade2,
      propriedadeOpcional: entity.propriedadeOpcional,
    );
  }

  /// Getter para converter de volta para Entity
  [Nome]Entity get toEntity => this;

  /// Conversão para Map (para APIs/persistência)
  Map<String, dynamic> get toMap {
    return {
      'propriedade1': propriedade1,
      'propriedade2': propriedade2.toJson,
      'propriedade_opcional': propriedadeOpcional,
    };
  }

  /// Conversão para JSON String (quando necessário)
  String get toJson => jsonEncode(toMap);

  /// Implementação do Equatable
  @override
  List<Object?> get props => [
    propriedade1,
    propriedade2,
    propriedadeOpcional,
  ];

  @override
  bool? get stringify => true;
}
```

---

## 📚 Exemplo Prático: UserModel

### Implementação Completa

```dart
import 'package:base_core/base_core.dart';

import '../../domain/entities/user_entity.dart';
import '../../domain/enums/account_person_type.dart';
import '../../domain/enums/user_gender_type.dart';
import 'address_model.dart';
import 'user_notification_preferences_model.dart';

class UserModel extends UserEntity with EquatableMixin {
  const UserModel({
    required super.id,
    required super.rg,
    required super.cpf,
    required super.name,
    required super.email,
    required super.phone,
    required super.birth,
    required super.gender,
    required super.address,
    required super.personType,
    required super.notificationPreferences,
  });

  /// Cria um UserModel a partir de dados da API/Database
  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      // Campos básicos com tratamento de nulos
      id: map['id']?.toString() ?? '',
      rg: map['rg']?.toString() ?? '',
      cpf: map['cpf']?.toString().replaceAll(RegExp(r'[^0-9]'), '') ?? '',
      name: map['name']?.toString() ?? '',
      email: map['email']?.toString() ?? '',
      phone: map['phone']?.toString() ?? '',
      
      // Enums com tratamento seguro
      gender: UserGenderType.fromJson(map['gender']),
      personType: AccountPersonType.fromJson(map['personType']),
      
      // Objetos complexos com validação
      address: AddressModel.fromMap(map['address'] ?? {}),
      notificationPreferences: UserNotificationPreferencesModel.fromMap(
        map['notification_preferences'] ?? {},
      ),
      
      // Data com parsing customizado e fallback
      birth: DateFormat.tryParseOrDateNow(
        map['birth_date'],
        pattern: 'yyyy-MM-dd',
      ),
    );
  }

  /// Cria um UserModel a partir de uma UserEntity
  factory UserModel.fromEntity(UserEntity entity) {
    return UserModel(
      id: entity.id,
      rg: entity.rg,
      cpf: entity.cpf,
      name: entity.name,
      birth: entity.birth,
      email: entity.email,
      phone: entity.phone,
      gender: entity.gender,
      address: entity.address,
      personType: entity.personType,
      notificationPreferences: entity.notificationPreferences,
    );
  }

  /// Converte o model de volta para entity (útil para testes e uso no domain)
  UserEntity get toEntity => this;

  /// Converte para Map para envio para API/Database
  Map<String, dynamic> get toMap {
    return {
      'id': id,
      'rg': rg,
      'name': name,
      'email': email,
      'phone': phone,
      'gender': gender.toJson,
      'personType': personType.toJson,
      'birth_date': birth.toIso8601String(),
      'cpf': cpf.replaceAll(RegExp(r'[^0-9]'), ''), // Remove formatação
      'address': AddressModel.fromEntity(address).toMap,
      'notification_preferences': UserNotificationPreferencesModel.fromEntity(
        notificationPreferences,
      ).toMap,
    };
  }

  /// Conversão para JSON string (quando necessário)
  String get toJson => jsonEncode(toMap);

  /// Factory para criar a partir de JSON string
  factory UserModel.fromJson(String jsonString) {
    final map = jsonDecode(jsonString) as Map<String, dynamic>;
    return UserModel.fromMap(map);
  }

  /// Implementação do Equatable para comparações
  @override
  List<Object?> get props => [
    id,
    rg,
    cpf,
    name,
    birth,
    email,
    phone,
    gender,
    address,
    personType,
    notificationPreferences,
  ];

  /// Habilita toString() automático do Equatable
  @override
  bool? get stringify => true;
}
```

---

## 🎨 Padrões de Implementação

### 1. Tratamento de Dados Nulos/Inválidos

```dart
factory ProductModel.fromMap(Map<String, dynamic> map) {
  return ProductModel(
    // String: valor padrão vazio
    id: map['id']?.toString() ?? '',
    name: map['name']?.toString() ?? 'Produto sem nome',
    
    // Números: parsing seguro
    price: double.tryParse(map['price']?.toString() ?? '0') ?? 0.0,
    quantity: int.tryParse(map['quantity']?.toString() ?? '0') ?? 0,
    
    // Booleanos: valor padrão explícito
    isActive: map['is_active'] ?? true,
    
    // Datas: parsing customizado
    createdAt: DateTime.tryParse(map['created_at'] ?? '') ?? DateTime.now(),
    
    // Listas: verificação de tipo
    tags: map['tags'] is List 
        ? List<String>.from(map['tags']) 
        : <String>[],
    
    // Objetos aninhados: verificação de nulo
    category: map['category'] != null 
        ? CategoryModel.fromMap(map['category'])
        : CategoryModel.empty(),
  );
}
```

### 2. Mapeamento de Campos com Nomes Diferentes

```dart
factory OrderModel.fromMap(Map<String, dynamic> map) {
  return OrderModel(
    // API usa 'order_id', entity usa 'id'
    id: map['order_id']?.toString() ?? '',
    
    // API usa 'customer_email', entity usa 'customerEmail'
    customerEmail: map['customer_email'] ?? '',
    
    // API usa 'total_amount', entity usa 'totalAmount'
    totalAmount: double.tryParse(map['total_amount']?.toString() ?? '0') ?? 0.0,
    
    // API usa 'created_timestamp', entity usa 'createdAt'
    createdAt: DateTime.fromMillisecondsSinceEpoch(
      map['created_timestamp'] ?? 0,
    ),
  );
}

Map<String, dynamic> get toMap {
  return {
    // Volta para o formato da API
    'order_id': id,
    'customer_email': customerEmail,
    'total_amount': totalAmount,
    'created_timestamp': createdAt.millisecondsSinceEpoch,
  };
}
```

### 3. Validação e Transformação de Dados

```dart
factory UserModel.fromMap(Map<String, dynamic> map) {
  // Limpeza e validação de CPF
  final rawCpf = map['cpf']?.toString() ?? '';
  final cleanCpf = rawCpf.replaceAll(RegExp(r'[^0-9]'), '');
  
  // Validação de email
  final email = map['email']?.toString() ?? '';
  final validEmail = _isValidEmail(email) ? email : '';
  
  // Normalização de nome
  final name = _normalizeName(map['name']?.toString() ?? '');
  
  return UserModel(
    cpf: cleanCpf,
    email: validEmail,
    name: name,
    // ... outros campos
  );
}

static String _normalizeName(String name) {
  return name
      .trim()
      .split(' ')
      .map((word) => word.isEmpty ? '' : 
           word[0].toUpperCase() + word.substring(1).toLowerCase())
      .where((word) => word.isNotEmpty)
      .join(' ');
}

static bool _isValidEmail(String email) {
  return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
}
```

---

## 🔧 Padrões Avançados

### Models com Relacionamentos

```dart
class OrderModel extends OrderEntity with EquatableMixin {
  OrderModel({
    required super.id,
    required super.customerId,
    required super.items,
    required super.status,
    required super.createdAt,
  });

  factory OrderModel.fromMap(Map<String, dynamic> map) {
    return OrderModel(
      id: map['id']?.toString() ?? '',
      customerId: map['customer_id']?.toString() ?? '',
      status: OrderStatus.fromJson(map['status']),
      createdAt: DateTime.tryParse(map['created_at'] ?? '') ?? DateTime.now(),
      
      // Lista de objetos complexos
      items: map['items'] is List
          ? (map['items'] as List)
              .map((item) => OrderItemModel.fromMap(item))
              .toList()
          : <OrderItemEntity>[],
    );
  }

  Map<String, dynamic> get toMap {
    return {
      'id': id,
      'customer_id': customerId,
      'status': status.toJson,
      'created_at': createdAt.toIso8601String(),
      'items': items.map((item) => 
        OrderItemModel.fromEntity(item).toMap
      ).toList(),
    };
  }

  @override
  List<Object?> get props => [id, customerId, items, status, createdAt];

  @override
  bool? get stringify => true;
}
```

### Models com Cache/Otimização

```dart
class ProductModel extends ProductEntity with EquatableMixin {
  ProductModel({
    required super.id,
    required super.name,
    required super.price,
    required super.category,
  });

  // Cache para conversões pesadas
  String? _cachedJson;
  Map<String, dynamic>? _cachedMap;

  factory ProductModel.fromMap(Map<String, dynamic> map) {
    final model = ProductModel(
      id: map['id']?.toString() ?? '',
      name: map['name']?.toString() ?? '',
      price: double.tryParse(map['price']?.toString() ?? '0') ?? 0.0,
      category: CategoryModel.fromMap(map['category'] ?? {}),
    );
    
    // Armazena o map original para evitar recomputação
    model._cachedMap = Map.from(map);
    return model;
  }

  Map<String, dynamic> get toMap {
    return _cachedMap ??= {
      'id': id,
      'name': name,
      'price': price,
      'category': CategoryModel.fromEntity(category).toMap,
    };
  }

  String get toJson {
    return _cachedJson ??= jsonEncode(toMap);
  }

  @override
  List<Object?> get props => [id, name, price, category];

  @override
  bool? get stringify => true;
}
```

---

## 🎯 Boas Práticas para Models

### ✅ Faça

```dart
// ✅ Use const constructor quando possível  
const UserModel({required super.id, required super.name});

// ✅ Estenda a entity correspondente
class UserModel extends UserEntity with EquatableMixin

// ✅ Use factory constructors descritivos
factory UserModel.fromMap(Map<String, dynamic> map)
factory UserModel.fromEntity(UserEntity entity)

// ✅ Trate dados nulos/inválidos com fallbacks seguros
price: double.tryParse(map['price']?.toString() ?? '0') ?? 0.0

// ✅ Use EquatableMixin para comparações automáticas
@override
List<Object?> get props => [id, name, email];

// ✅ Documente transformações complexas
/// Converte timestamp Unix para DateTime
createdAt: DateTime.fromMillisecondsSinceEpoch(map['timestamp'])

// ✅ Limpe e valide dados externos
cpf: cpf.replaceAll(RegExp(r'[^0-9]'), '')
```

### ❌ Não Faça

```dart
// ❌ Não adicione lógica de negócio nos models
bool get isVip => calculateVipStatus(); // deve ficar na entity

// ❌ Não ignore erros de parsing
price: double.parse(map['price']) // pode gerar exceção

// ❌ Não deixe de implementar toEntity 
// Sempre forneça uma forma de voltar para entity

// ❌ Não misture responsabilidades
void saveToDatabase() {} // responsabilidade do repository

// ❌ Não esqueça de tratar listas nulas
items: map['items'].map(...) // pode ser null

// ❌ Não use constructors não-const quando const é possível
UserModel({required super.id}); // deveria ser const

// ❌ Não implemente toJson/fromJson se não for necessário
// Apenas adicione quando realmente for usar
```

---

## 📋 Checklist para Criação de Models

- [ ] **Construtor const**: Usa `const` quando possível?
- [ ] **Herança**: Estende a entity correspondente?
- [ ] **EquatableMixin**: Implementado com props corretos?
- [ ] **fromMap**: Factory constructor implementado com tratamento de nulos?
- [ ] **fromEntity**: Factory constructor para conversão de entity?
- [ ] **toMap**: Método para serialização implementado?
- [ ] **toEntity**: Getter implementado (sempre implementar)?
- [ ] **Props**: Lista completa de propriedades no Equatable?
- [ ] **Stringify**: Habilitado para debug?
- [ ] **Validação**: Dados externos validados/limpos?
- [ ] **Nomenclatura**: Nome termina com `Model`?
- [ ] **Imports**: Apenas imports necessários incluídos?
- [ ] **JSON**: Métodos toJson/fromJson apenas quando necessário?

---

## 🚀 Exemplo de Uso Completo

```dart
void exemploDeUso() async {
  // 1. Recebendo dados da API
  final apiResponse = {
    'id': '123',
    'rg': '123456789',
    'cpf': '12345678901',
    'name': 'João da Silva',
    'email': 'joao@email.com',
    'phone': '11999999999',
    'birth_date': '1990-05-15',
    'gender': 'MALE',
    'personType': 'INDIVIDUAL',
    'address': {
      'street': 'Rua das Flores, 123',
      'city': 'São Paulo',
      'state': 'SP',
      'zipCode': '01234-567'
    },
    'notification_preferences': {
      'email_enabled': true,
      'sms_enabled': false
    },
  };

  // 2. Convertendo para model
  final userModel = UserModel.fromMap(apiResponse);

  // 3. Usando como entity e aplicando regras de negócio
  final userEntity = userModel.toEntity;
  if (userEntity.isAdult) {
    print('Usuário maior de idade');
  }

  if (userEntity.canReceiveNotifications()) {
    print('Pode enviar notificações para: ${userEntity.displayName}');
  }

  print('CPF formatado: ${userEntity.formattedCpf}');
  
  // 4. Modificando via entity copyWith
  final updatedEntity = userEntity.copyWith(
    email: 'novo.email@exemplo.com',
  );

  // 5. Convertendo de volta para model para persistência
  final updatedModel = UserModel.fromEntity(updatedEntity);

  // 6. Enviando para API
  final mapToSend = updatedModel.toMap;
  final jsonToSend = updatedModel.toJson; // Se necessário
  // await http.post('/api/users', body: jsonToSend);

  // 7. Comparando models
  final sameUser = UserModel.fromMap(apiResponse);
  print(userModel == sameUser); // true (graças ao EquatableMixin)
}
```

Esta estrutura garante que seus models sejam robustos, performáticos e mantenham a separação de responsabilidades entre domínio e infraestrutura.
