# Validators (Presentation)

## O que são?

Validators são **funções de validação** que verificam se os dados de entrada atendem aos critérios esperados. Eles são usados principalmente em formulários e inputs para garantir que os dados sejam válidos antes de serem processados.

## Responsabilidades

- Validar dados de entrada (campos de texto, dropdowns, etc.)
- Retornar mensagens de erro descritivas
- Não modificar dados (apenas validar)
- Ser reutilizáveis em diferentes contextos

## Estrutura

```dart
class FormValidators {
  // Validação de campo vazio
  static String? emptyField(String? input, [String fieldName = '']) {
    if (input == null || input.isEmpty) {
      return '''${fieldName.isEmpty ? 'Este campo' : fieldName} é obrigatório.''';
    }
    return null;
  }

  // Validação de dropdown vazio
  static String? emptyDropdown<T>(T? value, [String fieldName = '']) {
    if (value == null) {
      return '''${fieldName.isEmpty ? 'Selecione uma opção' : fieldName}''';
    }
    return null;
  }

  // Validação de email
  static String? invalidEmail(String? input) {
    if (input == null || input.isEmpty) return null;

    RegExp regex = RegExp(r'^[\w+.-]+(\.[\w+.-]+)*@[\w-]+(\.[\w-]+)+$');
    if (!regex.hasMatch(input)) {
      return 'E-mail inválido';
    }
    return null;
  }
}
```

## Características

### 1. Retorno Nullable

```dart
static String? emptyField(String? input, [String fieldName = '']) {
  if (input == null || input.isEmpty) {
    return 'Campo obrigatório';  // Retorna mensagem de erro
  }
  return null;  // Retorna null se válido
}
```

- Retorna `String?` (mensagem de erro ou null)
- `null` significa que o campo é válido
- `String` contém a mensagem de erro

### 2. Parâmetros Opcionais

```dart
static String? emptyField(String? input, [String fieldName = '']) {
  // fieldName é opcional e tem valor padrão ''
}
```

- Permite customizar mensagens de erro
- Torna validators mais flexíveis

### 3. Validação Condicional

```dart
static String? invalidEmail(String? input) {
  if (input == null || input.isEmpty) return null;  // Não valida se vazio

  RegExp regex = RegExp(r'^[\w+.-]+(\.[\w+.-]+)*@[\w-]+(\.[\w-]+)+$');
  if (!regex.hasMatch(input)) {
    return 'E-mail inválido';
  }
  return null;
}
```

- Valida apenas se o campo não estiver vazio
- Permite combinar com `emptyField` para campos obrigatórios

## Tipos de Validators

### 1. Validators de Campo Vazio

```dart
// Campo de texto obrigatório
FormValidators.emptyField(value, 'Nome')

// Para dropdowns, use emptyField com conversão para string
FormValidators.emptyField(yearSelected?.toString(), 'Selecione o ano')
```

### 2. Validators de Formato

```dart
// Email
FormValidators.invalidEmail(email)

// CPF
FormValidators.invalidCPF(cpf)

// CNPJ
FormValidators.invalidCNPJ(cnpj)

// Telefone
FormValidators.invalidPhone(phone)

// CEP
FormValidators.invalidZipCode(zipCode)
```

### 3. Validators de Tamanho

```dart
// Tamanho mínimo
FormValidators.invalidLength(input, 3)

// Tamanho mínimo e máximo
FormValidators.invalidLength(input, 3, max: 50)
```

### 4. Validators de Valor

```dart
// Valor monetário maior que
FormValidators.invalidGreaterThanCurrency(value, 100.0)

// Valor monetário menor que
FormValidators.invalidLessThanCurrency(value, 1000.0)

// Valor entre min e max
FormValidators.invalidGreaterThanAndLessThanOrEqualToCurrency(value, 100.0, 1000.0)
```

### 5. Validators de Data

```dart
// Data válida
FormValidators.invalidDate(date)

// Data menor que referência
FormValidators.invalidDateLessThan(date, DateTime.now())

// Data maior que referência
FormValidators.invalidDateGreaterThan(date, DateTime(2020, 1, 1))
```

### 6. Validators de Senha

```dart
// Senha forte (mínimo 8 caracteres, maiúscula, minúscula, número, especial)
FormValidators.invalidStrongPassword(password, 8)

// Contém número
FormValidators.invalidContainsNumber(password)

// Contém caractere especial
FormValidators.invalidSpecialCharacter(password)
```

### 7. Validators Bancários

```dart
// Agência bancária
FormValidators.invalidBankAgency(agency)

// Conta bancária
FormValidators.invalidBankAccount(account)

// Nome do banco
FormValidators.invalidBankName(bankName)
```

## Padrões de Uso

### Padrão Recomendado: Form com Listas de Validators

Este é o padrão usado em todo o projeto. Use para formulários com campos de texto, dropdowns e componentes customizados:

**Características:**

- Declare `_formKey = GlobalKey<FormState>()` no State
- Crie **listas de validators como getters** para cada campo
- Passe validators via parâmetro `validators` para os componentes
- Use `validateGranularly()` para validar e coletar todos os erros
- Mostre erros concatenados com `\n`

**Exemplo 1: Formulário com Dropdowns (Income Report)**

```dart
class _IncomeReportPageState extends State<IncomeReportPage> {
  final _formKey = GlobalKey<FormState>();

  int? yearSelected;
  UserEntity? _userSelected;

  UserEntity get _user => DM.i.get<SessionController>().user;

  // 1. Criar listas de validators como getters
  List<String? Function(String?)> get yearValidators => [
    (input) => FormValidators.emptyField(input, 'Ano de atuação'),
  ];

  List<String? Function(String?)> get userValidators => [
    if (_user.role.type == .admin)
      (input) => FormValidators.emptyField(input, 'Usuário'),
  ];

  void _onRequestIncomeReport() {
    // 2. Validar usando validateGranularly
    if (!(_formKey.currentState?.validate() ?? true)) {
      final errors = _formKey.currentState?.validateGranularly();
      return CustomSnackBar.toastShowMessage(
        context: context,
        message:
            errors?.map((e) => e.errorText).join('\n') ??
            'Verifique o preenchimento do formulário corretamente.',
        type: .error,
      );
    }

    // 3. Submeter com dados validados
    _submitForm();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Form(
        key: _formKey,
        child: Column(
          children: [
            // 4. Passar validators para componentes
            SelectUserDropdown(
              validators: userValidators,
              usersSelected: [_userSelected],
              onChanged: (user) => setState(() => _userSelected = user),
            ),

            CognaDropdown(
              validators: yearValidators,
              value: yearSelected,
              onChanged: (value) => setState(() => yearSelected = value),
              inputLabel: InputLabel(label: 'Ano de atuação', isRequired: true),
              items: years.map((y) {
                return CustomDropdownItem(value: y, label: '$y');
              }).toList(),
            ),
          ],
        ),
      ),
      bottomNavigationBar: BottomNavBarActions(
        onNext: _onRequestIncomeReport,
      ),
    );
  }
}
```

**Exemplo 2: Formulário com Campos de Texto (User Form)**

```dart
class _UserFormViewState extends State<_UserFormView> {
  late final TextEditingController nameTextController;
  late final TextEditingController emailTextController;
  late final TextEditingController cpfTextController;

  // 1. Criar listas de validators como getters
  List<String? Function(String?)> get nameValidators => [
    (input) => FormValidators.emptyField(input, 'Nome completo'),
    FormValidators.invalidFullName,
  ];

  List<String? Function(String?)> get cpfValidators => [
    (input) => FormValidators.emptyField(input, 'CPF'),
    FormValidators.invalidCPF,
  ];

  List<String? Function(String?)> get emailValidators => [
    (input) => FormValidators.emptyField(input, 'E-mail'),
    FormValidators.invalidEmail,
  ];

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 2. Passar validators para componentes
        CognaInputField(
          validators: nameValidators,
          controller: nameTextController,
          inputLabel: InputLabel(label: 'Nome completo', isRequired: true),
          onChanged: (input) {
            user = user.copyWith(name: input);
            widget.onChanged(user);
          },
        ),

        CognaInputField(
          validators: cpfValidators,
          controller: cpfTextController,
          inputLabel: InputLabel(label: 'CPF', isRequired: true),
          onChanged: (input) {
            user = user.copyWith(cpf: input);
            widget.onChanged(user);
          },
        ),

        CognaInputField(
          validators: emailValidators,
          controller: emailTextController,
          inputLabel: InputLabel(label: 'E-mail', isRequired: true),
          onChanged: (input) {
            user = user.copyWith(email: input);
            widget.onChanged(user);
          },
        ),
      ],
    );
  }
}
```

**Exemplo 3: Validação Condicional**

```dart
// Validators condicionais baseados em estado
List<String? Function(String?)> get distributorValidators => [
  (input) => FormValidators.emptyField(input, 'Distribuidor'),
];

List<String? Function(String?)> get roleValidators => [
  (input) => FormValidators.emptyField(input, 'Perfil'),
];

// Aplicar apenas se usuário for admin
List<String? Function(String?)> get userValidators => [
  if (_user.role.type == .admin)
    (input) => FormValidators.emptyField(input, 'Usuário'),
];
```

## Exemplo Completo 1: Income Report (Form com Dropdowns)

Este exemplo mostra o padrão recomendado com Form e listas de validators, ideal para dropdowns e validações condicionais:

```dart
class IncomeReportPage extends StatefulWidget {
  const IncomeReportPage({super.key});

  @override
  State<IncomeReportPage> createState() => _IncomeReportPageState();
}

class _IncomeReportPageState extends State<IncomeReportPage> {
  final _incomeReportController = DM.i.get<IncomeReportRequestController>();
  final _incomeYearsController = DM.i.get<IncomeReportYearsController>();
  final _formKey = GlobalKey<FormState>();

  DistributorEntity? _distributorSelected;
  UserEntity? _userSelected;
  int? yearSelected;

  UserEntity get _user => DM.i.get<SessionController>().user;

  // 1. Criar listas de validators como getters
  List<String? Function(String?)> get yearValidators => [
    (input) => FormValidators.emptyField(input, 'Ano de atuação'),
  ];

  List<String? Function(String?)> get userValidators => [
    if (_user.role.type == .admin)
      (input) => FormValidators.emptyField(input, 'Usuário'),
  ];

  @override
  void initState() {
    super.initState();
    _distributorSelected = _user.partnerCompany;
  }

  void _onRequestIncomeReport() {
    // 2. Validar usando validateGranularly
    if (!(_formKey.currentState?.validate() ?? true)) {
      final errors = _formKey.currentState?.validateGranularly();
      return CustomSnackBar.toastShowMessage(
        context: context,
        message:
            errors?.map((e) => e.errorText).join('\n') ??
            'Verifique o preenchimento do formulário corretamente.',
        type: .error,
      );
    }

    // 3. Submeter com dados validados
    _incomeReportController
        .requestIncomeReport(
          request: IncomeReportRequestEntity(
            affiliateId: _userSelected?.id ?? _user.id,
            distributorId: _distributorSelected?.id,
            year: yearSelected!,
          ),
        )
        .then((onValue) {
          if (_incomeReportController.hasError) {
            return CustomSnackBar.toastShowMessage(
              context: context,
              message: _incomeReportController.error!.message,
              type: .error,
            );
          }

          // Sucesso
          CustomSnackBar.toastShowMessage(
            context: context,
            message:
                'Relatório solicitado com sucesso! Você receberá por e-mail.',
            type: .success,
          );
        });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(
        title: 'Informe de rendimentos',
        centerTitle: !kIsWeb,
      ),
      body: CustomScrollContent(
        padding: .symmetric(
          horizontal: Spacing.sm.value,
          vertical: Spacing.md.value,
        ),
        child: SafeArea(
          child: Form(
            key: _formKey,
            child: Column(
              crossAxisAlignment: .stretch,
              children: [
                // Dropdown de distribuidor (apenas para admin)
                if (<UserRoleType>[.admin].contains(_user.role.type)) ...[
                  SelectDistributorDropdown(
                    key: ValueKey(_distributorSelected),
                    distributorsSelected: [_distributorSelected],
                    onChanged: (distributor) {
                      setState(() => _distributorSelected = distributor);
                      if (distributor != null) {
                        _incomeYearsController.fetchAvailableYears(
                          distributorId: distributor.id,
                        );
                      }
                    },
                  ),
                  Spacing.sm.vertical,
                ],

                // 4. Passar validators para componentes
                SelectUserDropdown(
                  validators: userValidators,
                  key: ValueKey(_userSelected),
                  usersSelected: [_userSelected],
                  onChanged: (user) => setState(() => _userSelected = user),
                ),
                Spacing.sm.vertical,

                CustomListenableBuilder(
                  controller: _incomeYearsController,
                  builder: (context, state, isLoading, hasError, error) {
                    return CognaDropdown(
                      validators: yearValidators,
                      key: ValueKey(yearSelected),
                      isEnabled: !isLoading,
                      onChanged: (value) => setState(() => yearSelected = value),
                      placeholder: 'Informe o ano de atuação',
                      inputLabel: InputLabel(
                        label: 'Ano de atuação',
                        isRequired: true,
                      ),
                      items: state.map((y) {
                        return CustomDropdownItem(value: y, label: '$y');
                      }).toList(),
                    );
                  },
                ),
              ],
            ),
          ),
        ),
      ),
      bottomNavigationBar: BottomNavBarActions(
        onNext: _onRequestIncomeReport,
        nextButtonText: 'Solicitar relatório',
      ),
    );
  }
}
```

## Exemplo Completo 2: Upsert User (Form com Campos de Texto)

Este exemplo mostra validação com Form e múltiplos campos de texto:

```dart
class UpsertUserPage extends StatefulWidget {
  const UpsertUserPage({super.key, this.userId = ''});

  final String userId;

  @override
  State<UpsertUserPage> createState() => _UpsertUserPageState();
}

class _UpsertUserPageState extends State<UpsertUserPage> {
  final _userController = DM.i.get<UserController>();
  final _formKey = GlobalKey<FormState>();

  Future<void> onUpsertUser() async {
    // Validar todos os campos do formulário
    if (!(_formKey.currentState?.validate() ?? true)) {
      final errors = _formKey.currentState?.validateGranularly();
      return CustomSnackBar.toastShowMessage(
        context: context,
        message: errors?.map((e) => e.errorText).join('\n') ??
                'Verifique o preenchimento do formulário corretamente.',
        type: .error,
      );
    }

    // Remover foco dos campos
    FocusScope.of(context).requestScopeFocus();

    // Submeter formulário
    if (widget.userId.isEmpty) {
      await _userController.createUser(_userController.data);
    } else {
      await _userController.updateUserById(
        data: _userController.data,
        id: widget.userId,
      );
    }

    // Verificar erro
    if (_userController.hasError) {
      return CustomSnackBar.toastShowMessage(
        context: context,
        message: _userController.error!.message,
        type: .error,
      );
    }

    // Sucesso
    onBack();
    return CustomSnackBar.toastShowMessage(
      context: context,
      message: '''Cadastro ${widget.userId.isEmpty ? 'criado' : 'atualizado'} com sucesso!''',
      type: .success,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(
        title: widget.userId.isEmpty ? 'Adicionar novo usuário' : 'Atualizar usuário',
      ),
      body: CustomScrollContent(
        child: Form(
          key: _formKey,
          child: _UserFormView(
            user: _userController.data,
            onChanged: (user) {
              _userController.data = user;
            },
          ),
        ),
      ),
      bottomNavigationBar: BottomNavBarActions(
        onNext: onUpsertUser,
        nextButtonText: widget.userId.isEmpty ? 'Criar usuário' : 'Atualizar usuário',
      ),
    );
  }
}

// Widget separado com os campos do formulário
class _UserFormView extends StatefulWidget {
  const _UserFormView({
    required this.user,
    required this.onChanged,
  });

  final UserEntity user;
  final Function(UserEntity user) onChanged;

  @override
  State<_UserFormView> createState() => _UserFormViewState();
}

class _UserFormViewState extends State<_UserFormView> {
  late final TextEditingController nameTextController;
  late final TextEditingController emailTextController;
  late final TextEditingController cpfTextController;
  late final TextEditingController phoneTextController;

  // 1. Criar listas de validators como getters
  List<String? Function(String?)> get nameValidators => [
    (input) => FormValidators.emptyField(input, 'Nome completo'),
    FormValidators.invalidFullName,
  ];

  List<String? Function(String?)> get cpfValidators => [
    (input) => FormValidators.emptyField(input, 'CPF'),
    FormValidators.invalidCPF,
  ];

  List<String? Function(String?)> get emailValidators => [
    (input) => FormValidators.emptyField(input, 'E-mail'),
    FormValidators.invalidEmail,
  ];

  List<String? Function(String?)> get phoneValidators => [
    (input) => FormValidators.emptyField(input, 'Telefone'),
    FormValidators.invalidPhone,
  ];

  @override
  void initState() {
    super.initState();
    nameTextController = TextEditingController(text: widget.user.name);
    emailTextController = TextEditingController(text: widget.user.email);
    cpfTextController = TextEditingController(
      text: Formatters.formatCpf(widget.user.cpf),
    );
    phoneTextController = TextEditingController(
      text: Formatters.formatPhone(widget.user.phone),
    );
  }

  @override
  void dispose() {
    nameTextController.dispose();
    emailTextController.dispose();
    cpfTextController.dispose();
    phoneTextController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 2. Passar validators para componentes
        CognaInputField(
          validators: nameValidators,
          controller: nameTextController,
          hintText: 'Informe o nome completo',
          inputLabel: InputLabel(label: 'Nome completo', isRequired: true),
          onChanged: (input) {
            final user = widget.user.copyWith(name: input);
            widget.onChanged(user);
          },
        ),

        Spacing.sm.vertical,

        CognaInputField(
          validators: cpfValidators,
          controller: cpfTextController,
          hintText: 'Informe o CPF',
          inputLabel: InputLabel(label: 'CPF', isRequired: true),
          onChanged: (input) {
            final user = widget.user.copyWith(cpf: input);
            widget.onChanged(user);
          },
        ),

        Spacing.sm.vertical,

        CognaInputField(
          validators: emailValidators,
          controller: emailTextController,
          hintText: 'Informe o e-mail',
          inputLabel: InputLabel(label: 'E-mail', isRequired: true),
          onChanged: (input) {
            final user = widget.user.copyWith(email: input);
            widget.onChanged(user);
          },
        ),

        Spacing.sm.vertical,

        CognaInputField(
          validators: phoneValidators,
          controller: phoneTextController,
          hintText: 'Informe o telefone',
          inputLabel: InputLabel(label: 'Telefone', isRequired: true),
          onChanged: (input) {
            final user = widget.user.copyWith(phone: input);
            widget.onChanged(user);
          },
        ),
      ],
    );
  }
}
```

## Benefícios

- **Reutilização**: Validators podem ser usados em múltiplos formulários
- **Consistência**: Mensagens de erro padronizadas
- **Testabilidade**: Fácil testar validators isoladamente
- **Manutenibilidade**: Mudanças em validações centralizadas
- **Clareza**: Código de validação separado da lógica de negócio
- **Flexibilidade**: Validação manual ou automática conforme necessidade

## Criando Validators Customizados

```dart
class FormValidators {
  // Validator customizado para código de produto
  static String? invalidProductCode(String? input) {
    if (input == null || input.isEmpty) return null;

    // Código deve ter formato: ABC-1234
    final regex = RegExp(r'^[A-Z]{3}-\d{4}$');
    if (!regex.hasMatch(input)) {
      return 'Código deve ter formato ABC-1234';
    }

    return null;
  }

  // Validator customizado para idade mínima
  static String? invalidMinimumAge(String? input, int minimumAge) {
    if (input == null || input.isEmpty) return null;

    final birthDate = DateFormat.fromString(input, format: 'dd/MM/yyyy');
    if (birthDate == null) return 'Data inválida';

    final age = DateTime.now().difference(birthDate).inDays ~/ 365;
    if (age < minimumAge) {
      return 'Idade mínima: $minimumAge anos';
    }

    return null;
  }
}
```

## Boas Práticas

1. **Sempre use Form com GlobalKey<FormState>**
2. **Crie listas de validators como getters** (não inline)
3. **Passe validators via parâmetro** para os componentes
4. **Use validateGranularly()** para coletar todos os erros
5. **Mensagens de erro claras e descritivas**
6. **Validação condicional** com `if` nos getters de validators
7. **Combine validators** em listas para validações complexas
8. **Teste validators isoladamente**
9. **Documente validators customizados**
10. **Sempre retorne null para valores válidos**

## Quando Usar Este Padrão

### Use Form com Listas de Validators para:

- Formulários com múltiplos campos (texto, dropdowns, etc.)
- Validação automática ao submeter
- Feedback visual em cada campo
- Validações condicionais (usando `if` nos getters)
- Combinar múltiplos validators por campo
- Coletar todos os erros de uma vez com `validateGranularly()`

### Vantagens:

- **Consistência**: Mesmo padrão em todo o projeto
- **Flexibilidade**: Validators condicionais com `if` nos getters
- **Clareza**: Validators organizados como getters
- **Reutilização**: Listas de validators podem ser compartilhadas
- **Feedback completo**: `validateGranularly()` retorna todos os erros de uma vez

## Próximos Passos

- [Controllers](./controllers.md) - Usar validators em controllers
- [Modules](./modules.md) - Organizar validators em módulos
- [Pages](./pages.md) - Integrar validators em páginas
