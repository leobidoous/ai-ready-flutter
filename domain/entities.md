# Domain Entities - Clean Architecture

## 📋 Visão Geral

As **Entities** representam as regras de negócio mais fundamentais e estáveis da aplicação. Elas contêm a lógica que é menos propensa a mudanças quando algo externo muda.

### 🎯 Propósito

- **Regras de Negócio Centrais**: Encapsulam as regras mais importantes e estáveis
- **Independência**: Não dependem de frameworks, UI, banco de dados ou agentes externos
- **Reutilização**: Podem ser usadas por qualquer camada da aplicação
- **Teste**: Fáceis de testar isoladamente

### 📍 Localização na Arquitetura

```
lib/
└── src/
    └── domain/
        └── entities/
            ├── user_entity.dart
            ├── product_entity.dart
            └── order_entity.dart
```

---

## 🏗️ Estrutura Base de uma Entity

### Regras Fundamentais

1. **Imutabilidade**: Entities devem ser imutáveis por padrão
2. **Construtor const**: Use `const` no construtor quando possível para performance
3. **Parâmetros Nomeados**: Sempre usar `required` e parâmetros nomeados
4. **Validação**: Validações básicas com `assert` no construtor quando necessário
5. **copyWith**: Método para criar cópias com alterações
6. **Regras de Negócio**: Métodos que encapsulem lógica de domínio
7. **Igualdade**: Implementar `==`, `hashCode` e `toString` quando necessário
8. **Sem Dependências**: Não importar nada além de outras entities do domain

### Template Base

```dart
class [Nome]Entity {
  const [Nome]Entity({
    required this.propriedade1,
    required this.propriedade2,
    // Propriedades opcionais por último
    this.propriedadeOpcional,
  });

  // Propriedades finais (imutabilidade)
  final TipoPrimario propriedade1;
  final TipoComplexo propriedade2;
  final TipoOpcional? propriedadeOpcional;

  // Método copyWith para imutabilidade
  [Nome]Entity copyWith({
    TipoPrimario? propriedade1,
    TipoComplexo? propriedade2,
    TipoOpcional? propriedadeOpcional,
  }) {
    return [Nome]Entity(
      propriedade1: propriedade1 ?? this.propriedade1,
      propriedade2: propriedade2 ?? this.propriedade2,
      propriedadeOpcional: propriedadeOpcional ?? this.propriedadeOpcional,
    );
  }

  // Métodos de negócio (regras de domínio)
  bool metodoDeNegocio() {
    // Lógica de negócio pura
    return true;
  }

  // Implementação de igualdade e hashCode quando necessário
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is [Nome]Entity &&
      runtimeType == other.runtimeType &&
      propriedade1 == other.propriedade1 &&
      propriedade2 == other.propriedade2 &&
      propriedadeOpcional == other.propriedadeOpcional;

  @override
  int get hashCode =>
      propriedade1.hashCode ^
      propriedade2.hashCode ^
      propriedadeOpcional.hashCode;

  @override
  String toString() => '[Nome]Entity(propriedade1: $propriedade1, propriedade2: $propriedade2)';
}
```

---

## 📚 Exemplo Prático: UserEntity

### Implementação Completa

```dart
import '../../../cogna_resale_core.dart' show AccountPersonType;
import '../enums/user_gender_type.dart';
import 'address_entity.dart';
import 'user_notification_preferences_entity.dart';

class UserEntity {
  const UserEntity({
    required this.id,
    required this.rg,
    required this.cpf,
    required this.name,
    required this.birth,
    required this.email,
    required this.phone,
    required this.gender,
    required this.address,
    required this.personType,
    required this.notificationPreferences,
  }) : assert(id != '', 'ID não pode ser vazio'),
       assert(cpf.length == 11, 'CPF deve ter exatamente 11 dígitos'),
       assert(email != '', 'Email não pode ser vazio');

  final String id;
  final String rg;
  final String cpf;
  final String name;
  final String email;
  final String phone;
  final DateTime birth;
  final AddressEntity address;
  final UserGenderType gender;
  final AccountPersonType personType;
  final UserNotificationPreferencesEntity notificationPreferences;

  /// Cria uma cópia da entidade com os campos alterados
  UserEntity copyWith({
    String? id,
    String? rg,
    String? cpf,
    String? name,
    String? email,
    String? phone,
    DateTime? birth,
    AddressEntity? address,
    UserGenderType? gender,
    AccountPersonType? personType,
    UserNotificationPreferencesEntity? notificationPreferences,
  }) {
    return UserEntity(
      id: id ?? this.id,
      rg: rg ?? this.rg,
      cpf: cpf ?? this.cpf,
      name: name ?? this.name,
      birth: birth ?? this.birth,
      email: email ?? this.email,
      phone: phone ?? this.phone,
      gender: gender ?? this.gender,
      address: address ?? this.address,
      personType: personType ?? this.personType,
      notificationPreferences: notificationPreferences ?? this.notificationPreferences,
    );
  }

  /// Regra de negócio: verificar se é maior de idade
  bool get isAdult {
    final now = DateTime.now();
    final eighteenYearsAgo = DateTime(now.year - 18, now.month, now.day);
    return birth.isBefore(eighteenYearsAgo) || birth.isAtSameMomentAs(eighteenYearsAgo);
  }

  /// Regra de negócio: nome formatado para exibição
  String get displayName {
    final nameParts = name.trim().split(' ');
    if (nameParts.length <= 2) return name;
    return '${nameParts.first} ${nameParts.last}';
  }

  /// Regra de negócio: validar se pode receber notificações
  bool canReceiveNotifications() {
    return notificationPreferences.emailEnabled || 
           notificationPreferences.smsEnabled;
  }

  /// Regra de negócio: CPF formatado para exibição
  String get formattedCpf {
    if (cpf.length != 11) return cpf;
    return '${cpf.substring(0, 3)}.${cpf.substring(3, 6)}.${cpf.substring(6, 9)}-${cpf.substring(9)}';
  }

  /// Regra de negócio: verificar se é pessoa jurídica
  bool get isLegalPerson => personType == AccountPersonType.company;

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is UserEntity &&
      runtimeType == other.runtimeType &&
      id == other.id &&
      rg == other.rg &&
      cpf == other.cpf &&
      name == other.name &&
      birth == other.birth &&
      email == other.email &&
      phone == other.phone &&
      gender == other.gender &&
      address == other.address &&
      personType == other.personType &&
      notificationPreferences == other.notificationPreferences;

  @override
  int get hashCode =>
      id.hashCode ^
      rg.hashCode ^
      cpf.hashCode ^
      name.hashCode ^
      birth.hashCode ^
      email.hashCode ^
      phone.hashCode ^
      gender.hashCode ^
      address.hashCode ^
      personType.hashCode ^
      notificationPreferences.hashCode;

  @override
  String toString() => 'UserEntity(id: $id, name: $name, email: $email)';
}
```

---

## 🎨 Boas Práticas para Entities

### ✅ Faça

```dart
// ✅ Use const constructor quando possível
const UserEntity({required this.name, required this.email});

// ✅ Use final para imutabilidade
final String name;

// ✅ Parâmetros nomeados e required
UserEntity({required this.name, required this.email});

// ✅ Use assert para validações básicas no construtor
const UserEntity({required this.email}) 
    : assert(email != '', 'Email é obrigatório');

// ✅ Métodos que expressam regras de negócio
bool get isVip => totalPurchases > 10000;

// ✅ copyWith para mutabilidade controlada
UserEntity copyWith({String? name}) => UserEntity(name: name ?? this.name);

// ✅ Implemente igualdade quando necessário para comparações
@override
bool operator ==(Object other) => /* implementação */;
```

### ❌ Não Faça

```dart
// ❌ Não use propriedades mutáveis
String name; // sem final

// ❌ Não use constructors não-const quando const é possível
UserEntity({required this.name}); // deveria ser const

// ❌ Não faça validações complexas/externas no construtor
UserEntity({required this.email}) {
  if (await emailExists(email)) throw ArgumentError(); // async no construtor
}

// ❌ Não importe dependências externas
import 'package:http/http.dart'; // dependência externa

// ❌ Não implemente lógica de infraestrutura
void saveToDatabase() {} // responsabilidade da camada infra

// ❌ Não use parâmetros posicionais
UserEntity(this.name, this.email); // sem nomes

// ❌ Não deixe de implementar igualdade em entities importantes
// (sem == e hashCode quando necessário)
```

---

## 🔧 Padrões Avançados

### Value Objects

Para tipos mais complexos, crie value objects:

```dart
class Email {
  const Email({required this.value}) 
      : assert(value != '', 'Email não pode ser vazio');

  final String value;

  bool get isValid {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value);
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) || 
      other is Email && other.value == value;

  @override
  int get hashCode => value.hashCode;

  @override
  String toString() => 'Email($value)';
}
```

### Entities com Comportamentos

```dart
class OrderEntity {
  const OrderEntity({
    required this.id,
    required this.items,
    required this.status,
    required this.createdAt,
  });

  final String id;
  final List<OrderItemEntity> items;
  final OrderStatus status;
  final DateTime createdAt;

  // Regra de negócio: calcular total
  double get totalAmount {
    return items.fold(0.0, (sum, item) => sum + item.totalPrice);
  }

  // Regra de negócio: verificar se pode ser cancelado
  bool canBeCancelled() {
    const cancelableStatuses = [OrderStatus.pending, OrderStatus.confirmed];
    return cancelableStatuses.contains(status);
  }

  // Regra de negócio: aplicar desconto
  OrderEntity applyDiscount({required double percentage}) {
    if (percentage < 0 || percentage > 100) {
      throw ArgumentError('Desconto deve estar entre 0 e 100%');
    }

    final discountedItems = items.map((item) => 
      item.applyDiscount(percentage: percentage)
    ).toList();

    return copyWith(items: discountedItems);
  }

  OrderEntity copyWith({
    String? id,
    List<OrderItemEntity>? items,
    OrderStatus? status,
    DateTime? createdAt,
  }) {
    return OrderEntity(
      id: id ?? this.id,
      items: items ?? this.items,
      status: status ?? this.status,
      createdAt: createdAt ?? this.createdAt,
    );
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is OrderEntity &&
      runtimeType == other.runtimeType &&
      id == other.id &&
      items == other.items &&
      status == other.status &&
      createdAt == other.createdAt;

  @override
  int get hashCode =>
      id.hashCode ^
      items.hashCode ^
      status.hashCode ^
      createdAt.hashCode;

  @override
  String toString() => 'OrderEntity(id: $id, status: $status, total: $totalAmount)';
}
```

---

## 📋 Checklist para Criação de Entities

- [ ] **Construtor const**: Usa `const` quando possível?
- [ ] **Imutabilidade**: Todas as propriedades são `final`?
- [ ] **Construtor**: Usa parâmetros nomeados com `required`?
- [ ] **Validações**: Validações básicas estão usando `assert`?
- [ ] **copyWith**: Método implementado corretamente?
- [ ] **Regras de Negócio**: Métodos expressos de forma clara?
- [ ] **Dependências**: Não importa nada da infraestrutura?
- [ ] **Nomenclatura**: Nome termina com `Entity`?
- [ ] **Igualdade**: `==`, `hashCode` e `toString` implementados quando necessário?
- [ ] **Documentação**: Métodos complexos estão documentados?

---

## 🚀 Exemplo de Uso

```dart
void exemploDeUso() {
  // Criação da entidade com const constructor
  const user = UserEntity(
    id: '123',
    name: 'João Silva',
    email: 'joao@email.com',
    cpf: '12345678901',
    rg: '123456789',
    phone: '11999999999',
    birth: DateTime(1990, 5, 15),
    gender: UserGenderType.male,
    address: AddressEntity(/* ... */),
    personType: AccountPersonType.individual,
    notificationPreferences: UserNotificationPreferencesEntity(/* ... */),
  );

  // Uso das regras de negócio
  if (user.isAdult) {
    print('Usuário é maior de idade');
  }

  if (user.canReceiveNotifications()) {
    print('Pode enviar notificações para: ${user.displayName}');
  }

  print('CPF formatado: ${user.formattedCpf}');
  print('É pessoa jurídica: ${user.isLegalPerson}');

  // Modificação via copyWith
  final updatedUser = user.copyWith(
    email: 'novo.email@email.com',
  );

  // Comparação entre entities
  final anotherUser = user.copyWith(name: 'Maria Silva');
  print('São o mesmo usuário? ${user == anotherUser}'); // false

  // Value objects
  const email = Email(value: 'test@example.com');
  if (email.isValid) {
    print('Email válido: $email');
  }
}
```

Esta estrutura garante que suas entities sejam robustas, testáveis e sigam os princípios do Clean Architecture.
