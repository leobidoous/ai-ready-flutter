# Domain Enums - Clean Architecture

## 📚 Visão Geral

Os **Enums** na camada de **Domain** definem **valores constantes e bem definidos** que representam estados, tipos ou categorias específicas do negócio. Eles garantem tipagem forte e consistência de dados através de toda a aplicação.

### 🎯 Princípios Fundamentais dos Enums

**O QUE os Enums DEFINEM:**
- ✅ **Valores Constantes**: Conjunto limitado e bem definido de opções
- ✅ **Tipagem Forte**: Substitui strings/ints por tipos específicos
- ✅ **Serialização Consistente**: Conversão padronizada para JSON/API
- ✅ **Nomes Legíveis**: Descrições human-readable para UI
- ✅ **Validação Automática**: Valores válidos garantidos pelo tipo

**O QUE os Enums NÃO FAZEM:**
- ❌ **Não contêm lógica complexa**: Apenas definição de valores
- ❌ **Não dependem de infraestrutura**: Zero dependências externas
- ❌ **Não mudam durante execução**: Valores imutáveis e fixos
- ❌ **Não quebram SOLID**: Seguem princípio da responsabilidade única
- ❌ **Não contêm regras de negócio**: Apenas categorização

### 🏗️ Localização e Estrutura

```
lib/src/domain/enums/
├── user_gender_type.dart
├── auth_provider_type.dart
├── order_status_type.dart
└── notification_type.dart
```

---

## 🔍 Anatomia de um Enum

### Estrutura Base

```dart
/// Enum que define [descrição dos valores]
/// 
/// Representa [contexto de uso] com valores bem definidos
/// para [finalidade específica].
enum [Nome]Type {
  [valor1](name: '[Nome Legível]', toJson: '[VALOR_API]'),
  [valor2](name: '[Nome Legível]', toJson: '[VALOR_API]'),
  [valor3](name: '[Nome Legível]', toJson: '[VALOR_API]');

  const [Nome]Type({required this.name, required this.toJson});

  /// Nome legível para exibição na UI
  final String name;
  
  /// Valor para serialização JSON/API
  final String toJson;

  /// Converte valor JSON/API para enum
  /// 
  /// [type] valor recebido da API ou JSON
  /// 
  /// Retorna o enum correspondente ou valor padrão se inválido
  static [Nome]Type fromJson(String? type) {
    return [Nome]Type.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => [Nome]Type.[valorPadrao],
    );
  }
}
```

### Elementos Essenciais

1. **Valores Enum**: Definição clara de cada opção disponível
2. **Propriedades Const**: name (UI) e toJson (API) como constantes
3. **Construtor Const**: Garantia de imutabilidade
4. **Método fromJson**: Deserialização segura com fallback
5. **Documentação Clara**: Propósito e contexto de uso

---

## 📚 Exemplos Práticos Reais

### 1. UserGenderType - Tipo de Gênero

```dart
/// Enum que define os tipos de gênero disponíveis para usuários
/// 
/// Representa as opções de gênero que podem ser selecionadas
/// no cadastro e perfil do usuário.
enum UserGenderType {
  male(name: 'Masculino', toJson: 'M'),
  female(name: 'Feminino', toJson: 'F'),
  other(name: 'Outro', toJson: 'NI');

  const UserGenderType({required this.name, required this.toJson});

  /// Nome legível para exibição na UI
  final String name;
  
  /// Valor para serialização JSON/API
  final String toJson;

  /// Converte valor JSON/API para enum de gênero
  /// 
  /// [type] valor recebido da API ('M', 'F', 'NI')
  /// 
  /// Retorna o enum correspondente ou 'other' se inválido
  static UserGenderType fromJson(String? type) {
    return UserGenderType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => UserGenderType.other,
    );
  }
}
```

### 2. AuthProviderType - Provedor de Autenticação

```dart
/// Enum que define os provedores de autenticação disponíveis
/// 
/// Representa os diferentes métodos de autenticação que o usuário
/// pode escolher para fazer login no sistema.
enum AuthProviderType {
  whatsapp(name: 'WhatsApp', toJson: 'WHATSAPP'),
  email(name: 'E-mail', toJson: 'EMAIL'),
  sms(name: 'SMS', toJson: 'SMS');

  const AuthProviderType({required this.name, required this.toJson});

  /// Nome legível para exibição na UI
  final String name;
  
  /// Valor para serialização JSON/API
  final String toJson;

  /// Converte valor JSON/API para enum de provedor
  /// 
  /// [type] valor recebido da API ('WHATSAPP', 'EMAIL', 'SMS')
  /// 
  /// Retorna o enum correspondente ou 'whatsapp' se inválido
  static AuthProviderType fromJson(String? type) {
    return AuthProviderType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => AuthProviderType.whatsapp,
    );
  }
}
```

### Características dos Exemplos Reais

✅ **Padrão consistente:**
- Propriedades `name` (UI) e `toJson` (API)
- Construtor const com parâmetros nomeados obrigatórios
- Método estático `fromJson` com fallback seguro

✅ **Nomes descritivos:**
- Sufixo `Type` para indicar que é um enum
- Valores em camelCase para Dart
- Documentação clara do propósito

✅ **Serialização robusta:**
- `toUpperCase()` para normalização
- `orElse` com valor padrão sensato
- Mapeamento direto entre enum e API

---

## 🎨 Padrões de Implementação

### 1. Enum Simples (Estados)
```dart
enum OrderStatusType {
  pending(name: 'Pendente', toJson: 'PENDING'),
  processing(name: 'Processando', toJson: 'PROCESSING'),
  shipped(name: 'Enviado', toJson: 'SHIPPED'),
  delivered(name: 'Entregue', toJson: 'DELIVERED'),
  cancelled(name: 'Cancelado', toJson: 'CANCELLED');

  const OrderStatusType({required this.name, required this.toJson});

  final String name;
  final String toJson;

  static OrderStatusType fromJson(String? type) {
    return OrderStatusType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => OrderStatusType.pending,
    );
  }
}
```

### 2. Enum com Propriedades Adicionais
```dart
enum NotificationPriorityType {
  low(name: 'Baixa', toJson: 'LOW', color: 0xFF4CAF50, order: 1),
  medium(name: 'Média', toJson: 'MEDIUM', color: 0xFFFF9800, order: 2),
  high(name: 'Alta', toJson: 'HIGH', color: 0xFFF44336, order: 3),
  urgent(name: 'Urgente', toJson: 'URGENT', color: 0xFF9C27B0, order: 4);

  const NotificationPriorityType({
    required this.name,
    required this.toJson,
    required this.color,
    required this.order,
  });

  final String name;
  final String toJson;
  final int color;
  final int order;

  static NotificationPriorityType fromJson(String? type) {
    return NotificationPriorityType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => NotificationPriorityType.low,
    );
  }

  /// Compara prioridades para ordenação
  bool isHigherThan(NotificationPriorityType other) {
    return order > other.order;
  }
}
```

### 3. Enum com Métodos Úteis
```dart
enum PaymentMethodType {
  creditCard(name: 'Cartão de Crédito', toJson: 'CREDIT_CARD', isInstant: true),
  debitCard(name: 'Cartão de Débito', toJson: 'DEBIT_CARD', isInstant: true),
  pix(name: 'PIX', toJson: 'PIX', isInstant: true),
  bankSlip(name: 'Boleto', toJson: 'BANK_SLIP', isInstant: false),
  bankTransfer(name: 'Transferência', toJson: 'BANK_TRANSFER', isInstant: false);

  const PaymentMethodType({
    required this.name,
    required this.toJson,
    required this.isInstant,
  });

  final String name;
  final String toJson;
  final bool isInstant;

  static PaymentMethodType fromJson(String? type) {
    return PaymentMethodType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => PaymentMethodType.creditCard,
    );
  }

  /// Retorna apenas métodos de pagamento instantâneo
  static List<PaymentMethodType> get instantMethods {
    return PaymentMethodType.values.where((e) => e.isInstant).toList();
  }

  /// Retorna apenas métodos de pagamento não instantâneo
  static List<PaymentMethodType> get nonInstantMethods {
    return PaymentMethodType.values.where((e) => !e.isInstant).toList();
  }
}
```

### 4. Enum com Validação Customizada
```dart
enum DocumentType {
  cpf(name: 'CPF', toJson: 'CPF', length: 11, mask: '###.###.###-##'),
  cnpj(name: 'CNPJ', toJson: 'CNPJ', length: 14, mask: '##.###.###/####-##'),
  rg(name: 'RG', toJson: 'RG', length: 9, mask: '##.###.###-#'),
  passport(name: 'Passaporte', toJson: 'PASSPORT', length: 8, mask: '########');

  const DocumentType({
    required this.name,
    required this.toJson,
    required this.length,
    required this.mask,
  });

  final String name;
  final String toJson;
  final int length;
  final String mask;

  static DocumentType fromJson(String? type) {
    return DocumentType.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => DocumentType.cpf,
    );
  }

  /// Valida se o documento tem o comprimento correto
  bool isValidLength(String document) {
    final cleanDocument = document.replaceAll(RegExp(r'[^0-9]'), '');
    return cleanDocument.length == length;
  }

  /// Aplica a máscara no documento
  String applyMask(String document) {
    final cleanDocument = document.replaceAll(RegExp(r'[^0-9]'), '');
    if (cleanDocument.length > length) {
      return cleanDocument.substring(0, length);
    }
    
    String masked = mask;
    for (int i = 0; i < cleanDocument.length; i++) {
      masked = masked.replaceFirst('#', cleanDocument[i]);
    }
    return masked.replaceAll('#', '');
  }
}
```

---

## 📋 Template para Enums

### Estrutura Básica

```dart
/// Enum que define [descrição dos valores possíveis]
/// 
/// Representa [contexto de uso] com valores bem definidos
/// para [finalidade específica na aplicação].
enum [Nome]Type {
  [valor1](name: '[Nome para UI]', toJson: '[VALOR_API]'),
  [valor2](name: '[Nome para UI]', toJson: '[VALOR_API]'),
  [valorN](name: '[Nome para UI]', toJson: '[VALOR_API]');

  const [Nome]Type({required this.name, required this.toJson});

  /// Nome legível para exibição na interface do usuário
  final String name;
  
  /// Valor usado para serialização em JSON e comunicação com API
  final String toJson;

  /// Converte valor recebido da API/JSON para enum
  /// 
  /// [type] valor recebido (geralmente string da API)
  /// 
  /// Retorna o enum correspondente ou valor padrão se inválido
  static [Nome]Type fromJson(String? type) {
    return [Nome]Type.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => [Nome]Type.[valorPadrao],
    );
  }
}
```

### Convenções de Enum

**Nomenclatura:**
- Enum: `[Nome]Type` (sufixo Type para clareza)
- Valores: camelCase para Dart convention
- Propriedades: `name` (UI) e `toJson` (API)

**Estrutura:**
- Construtor const com parâmetros required
- Propriedades final para imutabilidade
- Método estático fromJson com fallback
- Documentação clara do propósito

**Serialização:**
- toJson: valores em UPPER_CASE para APIs
- fromJson: normalização com toUpperCase()
- orElse: sempre com valor padrão sensato

---

## 📋 Checklist para Enums

### Checklist de Criação ✅

**Estrutura do Enum:**
- [ ] Localizado em `lib/src/domain/enums/`
- [ ] Nome seguindo padrão `[Nome]Type`
- [ ] Valores em camelCase (padrão Dart)
- [ ] Construtor const com parâmetros required
- [ ] Propriedades final para imutabilidade

**Propriedades Obrigatórias:**
- [ ] `name`: String legível para UI
- [ ] `toJson`: String para serialização API
- [ ] Propriedades adicionais quando necessário
- [ ] Todas as propriedades como final

**Serialização:**
- [ ] Método estático `fromJson(String? type)`
- [ ] Normalização com `toUpperCase()`
- [ ] `orElse` com valor padrão apropriado
- [ ] Tratamento de valores null/inválidos

**Documentação:**
- [ ] Descrição clara do propósito do enum
- [ ] Contexto de uso na aplicação
- [ ] Documentação das propriedades
- [ ] Exemplos quando necessário

**Qualidade:**
- [ ] Valores bem definidos e finitos
- [ ] Nomes descritivos para cada valor
- [ ] Mapeamento consistente API ↔ Enum
- [ ] Testes unitários para serialização

---

## 🎯 Diretrizes para Enums

### ✅ Boas Práticas

```dart
// ✅ Nomenclatura clara e consistente
enum UserAccountType {
  individual(name: 'Pessoa Física', toJson: 'INDIVIDUAL'),
  business(name: 'Pessoa Jurídica', toJson: 'BUSINESS'),
}

// ✅ Propriedades úteis para o domínio
enum SubscriptionPlanType {
  basic(name: 'Básico', toJson: 'BASIC', price: 9.90, features: 10),
  premium(name: 'Premium', toJson: 'PREMIUM', price: 19.90, features: 50),
  enterprise(name: 'Enterprise', toJson: 'ENTERPRISE', price: 49.90, features: -1),
}

// ✅ Fallback sensato no fromJson
static UserAccountType fromJson(String? type) {
  return UserAccountType.values.firstWhere(
    (e) => e.toJson == type?.toUpperCase(),
    orElse: () => UserAccountType.individual, // padrão mais comum
  );
}

// ✅ Métodos utilitários quando apropriado
static List<PaymentMethodType> get digitalMethods {
  return values.where((e) => e.isDigital).toList();
}
```

### ❌ Evitar

```dart
// ❌ Nomes genéricos ou confusos
enum Type1 { value1, value2, value3 }
enum UserStuff { thing1, thing2 }

// ❌ Valores mutáveis
enum StatusType {
  active(name: 'Ativo', toJson: 'ACTIVE');
  
  StatusType({required this.name, required this.toJson});
  
  String name; // sem final - mutável
  String toJson; // sem final - mutável
}

// ❌ fromJson sem tratamento de erro
static StatusType fromJson(String type) {
  return StatusType.values.firstWhere((e) => e.toJson == type); // pode gerar exception
}

// ❌ Valores de API inconsistentes
enum StatusType {
  active(name: 'Ativo', toJson: 'ativo'),      // minúscula
  inactive(name: 'Inativo', toJson: 'INACTIVE'), // maiúscula
  pending(name: 'Pendente', toJson: 'Pend'),     // abreviação
}
```

---

## 🚀 Uso em Entities e Models

### Em Entities
```dart
class UserEntity {
  const UserEntity({
    required this.id,
    required this.name,
    required this.gender,
    // ...
  });

  final String id;
  final String name;
  final UserGenderType gender; // Enum como tipo

  // Métodos que usam o enum
  bool get isMale => gender == UserGenderType.male;
  bool get isFemale => gender == UserGenderType.female;
}
```

### Em Models
```dart
class UserModel extends UserEntity with EquatableMixin {
  // Serialização para API
  Map<String, dynamic> get toMap => {
    'id': id,
    'name': name,
    'gender': gender.toJson, // Conversão para API
    // ...
  };

  // Deserialização da API
  static UserModel fromMap(Map<String, dynamic> map) {
    return UserModel(
      id: map['id'],
      name: map['name'],
      gender: UserGenderType.fromJson(map['gender']), // Conversão segura
      // ...
    );
  }
}
```

Esta estrutura garante que os Enums sejam **type-safe**, **bem documentados** e **fáceis de usar** em toda a aplicação! 🎯
