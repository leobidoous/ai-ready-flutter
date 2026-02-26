# Documentação de Export Standardization no Guidance

**Date**: 2026-02-23
**Type**: Documentation
**Status**: Completed

## Summary

Adicionada documentação completa sobre a padronização de exports no arquivo `guidance.md`. Esta documentação estabelece regras e padrões para estruturação hierárquica e recursiva de exports em todos os pacotes do projeto.

## Files Modified

- `.kiro/steering/guidance.md` - Adicionada seção "Export Standardization"

## Changes

### Nova Seção: Export Standardization

Adicionada seção completa com as seguintes subseções:

1. **Export Pattern Rules** - Regras fundamentais para estrutura de exports
2. **Export File Naming Convention** - Convenção de nomenclatura
3. **Hierarchical Structure Example** - Exemplo visual da estrutura
4. **Export File Content Pattern** - Padrões de conteúdo para diferentes tipos de pastas
5. **Usage Examples** - Exemplos de uso dos imports
6. **Conflict Resolution** - Como resolver conflitos de nomes
7. **Benefits** - Benefícios da padronização
8. **Maintenance Workflow** - Fluxo de trabalho para manutenção
9. **Enforcement** - Regras de aplicação

### Regras Estabelecidas

#### Regras Principais

1. Cada pasta DEVE ter um arquivo de export com o nome da pasta
2. Arquivos de export DEVEM exportar todos os arquivos da pasta
3. Arquivos de export DEVEM exportar todas as subpastas via seus exports
4. Estrutura DEVE ser recursiva do mais profundo ao mais raso
5. Export raiz DEVE exportar `src/src.dart`

#### Padrões de Conteúdo

- **Pasta folha** (sem subpastas): Exporta apenas arquivos
- **Pasta branch** (com subpastas): Exporta apenas subpastas
- **Pasta mista** (subpastas + arquivos): Exporta ambos
- **Export src**: Exporta todas as camadas
- **Export raiz**: Exporta pacotes externos + `src/src.dart`

### Exemplos Práticos

A documentação inclui exemplos práticos de:

- Como estruturar exports em diferentes níveis
- Como importar em diferentes granularidades
- Como resolver conflitos de nomes
- Como adicionar/remover arquivos e pastas
- Referência à implementação real no paypay-core

### Benefícios Documentados

1. Imports organizados e hierárquicos
2. Flexibilidade de importação
3. Facilidade de descoberta
4. Manutenibilidade simplificada
5. Performance com tree-shaking
6. Consistência entre pacotes

## Architecture Impact

- **Camada**: Documentation (Steering)
- **Escopo**: Todos os pacotes do projeto
- **Breaking Changes**: Não - apenas documentação
- **Novos Padrões**: Estabelece padrão obrigatório para exports

## Usage

Esta documentação será automaticamente incluída no contexto de todas as interações com a IA (devido a `inclusion: always` no frontmatter do guidance.md).

Desenvolvedores e IA devem seguir estes padrões ao:

- Criar novos pacotes
- Adicionar novos arquivos
- Criar novas pastas
- Refatorar estrutura de código

## Related Documentation

- `paypay-core/.kiro/changelogs/2026-02-23_export-standardization.md` - Implementação real no paypay-core
- `.kiro/steering/guidance.md` - Arquivo de guidance atualizado

## Enforcement

- AI assistants DEVEM seguir este padrão automaticamente
- Code reviews DEVEM verificar conformidade
- Novos pacotes DEVEM implementar esta estrutura
- Pacotes existentes DEVEM ser migrados gradualmente
