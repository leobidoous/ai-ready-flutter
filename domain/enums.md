# Enums - Domain Layer

## 📋 O que são Enums?

**Enums** são valores constantes e bem definidos do domínio que representam um conjunto fixo de opções. Eles garantem tipagem forte, evitam "strings mágicas" e tornam o código mais seguro e legível.

### 🎯 Responsabilidades

**✅ O que Enums FAZEM:**

- Definem **valores constantes** do domínio (status, tipos, categorias)
- Garantem **tipagem forte** evitando strings arbitrárias
- Fornecem **serialização/deserialização** consistente (toJson/fromJson)
- Carregam **metadados úteis** (nome legível, cores, ícones, etc.)
- Implementam **valores padrão** seguros via `orElse`

**❌ O que Enums NÃO FAZEM:**

- Não contêm lógica de negócio complexa
- Não dependem de frameworks externos (exceto tipos básicos como Color)
- Não fazem comunicação com APIs ou databases

---

## 🏗️ Exemplo Completo: UserStatus

Este exemplo demonstra **todas as melhores práticas** de Enums modernos no Dart:

```dart
import 'package:flutter/material.dart' show Color, Colors;

enum UserStatus {
  all(name: 'Todos', toJson: '', color: Colors.grey),
  active(name: 'Ativo', toJson: 'ACTIVE', color: Colors.green),
  disabled(name: 'Desativado', toJson: 'DISABLED', color: Colors.grey),
  deleted(name: 'Removido', toJson: 'DELETED', color: Colors.red);

  const UserStatus({
    required this.name,
    required this.toJson,
    required this.color,
  });

  // Metadados do enum
  final String name;      // Nome legível para UI
  final String toJson;    // Valor para serialização
  final Color color;      // Metadado visual

  // Deserialização segura com valor padrão
  static UserStatus fromJson(String? type) {
    return UserStatus.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => UserStatus.active,
    );
  }
}
```

### 🔑 Elementos Essenciais

1. **Enhanced Enum** - Dart moderno permite campos e métodos em enums
2. **Constructor `const`** - Otimização e imutabilidade
3. **Campos `final`** - Metadados imutáveis (name, toJson, color, etc.)
4. **Método `fromJson`** - Deserialização segura com `orElse`
5. **Exhaustiveness** - Compilador garante que todos os casos sejam tratados

---

## 🎯 Melhores Práticas Modernas

### 1. **Exhaustiveness Checking (Switch Expressions)**

```dart
// ✅ Dart moderno - Compilador garante todos os casos
String getMessage(UserStatus status) {
  return switch (status) {
    UserStatus.active => 'Usuário ativo',
    UserStatus.disabled => 'Usuário desativado',
    UserStatus.deleted => 'Usuário removido',
    UserStatus.all => 'Todos os usuários',
    // Se adicionar novo valor, compilador exige tratamento aqui!
  };
}

// ✅ Pattern matching com when
String getDetailedMessage(UserStatus status) {
  return switch (status) {
    UserStatus.active => 'Pode acessar o sistema',
    UserStatus.disabled => 'Acesso temporariamente bloqueado',
    UserStatus.deleted => 'Conta removida permanentemente',
    UserStatus.all => 'Filtro para visualização',
  };
}
```

**Benefício:** Se você adicionar um novo valor ao enum (ex: `suspended`), o compilador vai **forçar** você a tratar esse caso em todos os switches, evitando bugs!

### 2. **Métodos no Enum**

```dart
enum UserStatus {
  all(name: 'Todos', toJson: '', color: Colors.grey),
  active(name: 'Ativo', toJson: 'ACTIVE', color: Colors.green),
  disabled(name: 'Desativado', toJson: 'DISABLED', color: Colors.grey),
  deleted(name: 'Removido', toJson: 'DELETED', color: Colors.red);

  const UserStatus({
    required this.name,
    required this.toJson,
    required this.color,
  });

  final String name;
  final String toJson;
  final Color color;

  // Métodos auxiliares
  bool get canLogin => this == UserStatus.active;
  bool get isRemoved => this == UserStatus.deleted;
  bool get needsAction => this == UserStatus.disabled;

  static UserStatus fromJson(String? type) {
    return UserStatus.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => UserStatus.active,
    );
  }
}
```

### 3. **Serialização Segura**

```dart
// Em uma Entity
class UserEntity {
  const UserEntity({
    required this.name,
    required this.status,
  });

  final String name;
  final UserStatus status;
}

// Serialização (Entity → JSON)
final json = {
  'name': user.name,
  'status': user.status.toJson,  // 'ACTIVE'
};

// Deserialização (JSON → Entity)
final user = UserEntity(
  name: json['name'],
  status: UserStatus.fromJson(json['status']),  // Sempre retorna valor válido
);
```

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo        | Padrão                                     | Exemplo                                     |
| ----------- | ------------------------------------------ | ------------------------------------------- |
| **Enum**    | `{Nome}Type` ou `{Nome}Status`             | `UserGenderType`, `UserStatus`              |
| **Arquivo** | `{nome}_type.dart` ou `{nome}_status.dart` | `user_gender_type.dart`, `user_status.dart` |
| **Valores** | `camelCase`                                | `active`, `inProgress`, `waitingFor`        |

### ✅ Estrutura Padrão

```dart
enum UserStatus {
  // 1. Valores com metadados
  active(name: 'Ativo', toJson: 'ACTIVE', color: Colors.green),
  disabled(name: 'Desativado', toJson: 'DISABLED', color: Colors.grey);

  // 2. Constructor const
  const UserStatus({
    required this.name,
    required this.toJson,
    required this.color,
  });

  // 3. Campos finais
  final String name;
  final String toJson;
  final Color color;

  // 4. Método fromJson (obrigatório)
  static UserStatus fromJson(String? type) {
    return UserStatus.values.firstWhere(
      (e) => e.toJson == type?.toUpperCase(),
      orElse: () => UserStatus.active,
    );
  }

  // 5. Métodos auxiliares (opcional)
  bool get isActive => this == UserStatus.active;
}
```

---

## 🔧 Método fromJson - Deserialização Segura

### ✅ Padrão Recomendado

```dart
static UserStatus fromJson(String? type) {
  return UserStatus.values.firstWhere(
    (e) => e.toJson == type?.toUpperCase(),
    orElse: () => UserStatus.active,  // ⚠️ Sempre defina um padrão seguro
  );
}
```

### 🎯 Características Importantes

1. **Nullable (`String?`)** - Aceita valores nulos da API
2. **Case insensitive** - Normaliza com `toUpperCase()` ou `toLowerCase()`
3. **Valor padrão** - `orElse` garante retorno válido (nunca lança exceção)
4. **Seguro** - Sempre retorna um enum válido, mesmo com dados inválidos

---

## 🎯 Exemplos de Uso

### Comparações e Lógica

```dart
// Comparação direta
if (user.status == UserStatus.active) {
  print('Usuário pode acessar');
}

// Usando getters auxiliares
if (user.status.canLogin) {
  allowAccess();
}

// Switch expression (exhaustive)
final message = switch (user.status) {
  UserStatus.active => 'Bem-vindo!',
  UserStatus.disabled => 'Conta suspensa',
  UserStatus.deleted => 'Conta removida',
  UserStatus.all => 'Visualizando todos',
};
```

### Usando Metadados na UI

```dart
// Acessar nome legível
Text(user.status.name);  // 'Ativo'

// Usar cor do status
Container(
  decoration: BoxDecoration(
    color: user.status.color,
    borderRadius: BorderRadius.circular(8),
  ),
  child: Text(user.status.name),
);

// Listar todos os valores (ex: dropdown)
DropdownButton<UserStatus>(
  items: UserStatus.values.map((status) {
    return DropdownMenuItem(
      value: status,
      child: Row(
        children: [
          Icon(Icons.circle, color: status.color, size: 12),
          SizedBox(width: 8),
          Text(status.name),
        ],
      ),
    );
  }).toList(),
  onChanged: (value) => setState(() => selectedStatus = value),
);
```

---

## 📋 Checklist de Implementação

Ao criar um novo Enum, verifique:

- [ ] **Nomenclatura** segue padrão (`{Nome}Type` ou `{Nome}Status`)
- [ ] **Constructor `const`** implementado
- [ ] **Campo `name`** (nome legível) presente
- [ ] **Campo `toJson`** (serialização) presente
- [ ] **Método `fromJson`** com `orElse` e valor padrão seguro
- [ ] **Case handling** consistente com API (upper/lower)
- [ ] **Metadados extras** (color, icon) quando necessário para UI
- [ ] **Métodos auxiliares** (getters) quando úteis
- [ ] **Todos os switches** usam pattern matching para exhaustiveness

---

## 🚀 Benefícios dos Enums Modernos

### ✅ Exhaustiveness Checking

```dart
// Adicionar novo valor ao enum
enum UserStatus {
  active(...),
  disabled(...),
  deleted(...),
  suspended(...),  // ⬅️ Novo valor adicionado
}

// ⚠️ Compilador vai FORÇAR você a atualizar todos os switches!
String getMessage(UserStatus status) {
  return switch (status) {
    UserStatus.active => '...',
    UserStatus.disabled => '...',
    UserStatus.deleted => '...',
    // ❌ ERRO: Missing case: UserStatus.suspended
  };
}
```

### ✅ Segurança de Tipo

```dart
// ❌ String - Qualquer valor aceito (perigoso)
final String status = 'ACTIVO';  // Typo não detectado!

// ✅ Enum - Apenas valores válidos
final UserStatus status = UserStatus.active;  // Seguro e tipado
```

### ✅ Refatoração Segura

- Renomear um valor atualiza todas as referências automaticamente
- IDE mostra todos os usos do enum
- Autocomplete sugere apenas valores válidos

---

## 🔗 Próximos Passos

1. **[Usar em Entities](./entities.md)** - Substituir Strings por Enums
2. **[Criar Failures](./failures.md)** - Tipos de erro específicos do domínio
3. **[Definir Repository Interfaces](./i_repositories.md)** - Contratos de acesso aos dados

---

_Enums modernos do Dart garantem segurança em tempo de compilação. Use exhaustiveness checking para evitar bugs ao adicionar novos valores._
