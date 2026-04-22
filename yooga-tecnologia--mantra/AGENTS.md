
# CONTEXTO @YOOGA-TECNOLOGIA - MANTRA DESIGN SYSTEM

## Princípios Fundamentais

- **DRY**: Não se repetir, centralizar lógica comum em tokens e utilities
- **SOLID**: Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
- **KISS**: Manter simples, fazer o mais simples que funcione
- **YAGNI**: Implementar apenas o necessário agora
- **Clean Code**: Código autoexplicativo, sem comentários redundantes
- **Consistência**: Seguir padrões estabelecidos no projeto

## Regras Gerais

### Comunicação

- Responder **SEMPRE** em PT-BR
- Não se desculpar ou agradecer
- Comunicar de forma clara, direta, sem floreios
- Fornecer respostas concisas e relevantes

### Processo

- **Nunca quebrar funcionalidades existentes**
- Buscar menor diff possível
- Mudanças incrementais, arquivo por arquivo
- Documentar mudanças nas mensagens de commit
- Solicitar print do resultado e fornecer análise detalhada
- Implementar testes unitários quando criar funcionalidades novas

### Qualidade de Código

- **EVITAR** adicionar comentários redundantes ao código
- Preservar estruturas de código existentes
- Verificar informações antes de fazer mudanças
- Nunca adicionar placeholders ou TODOs
- Entregar código pronto para produção

## Estrutura do Projeto Mantra

### Arquitetura

```md
src/
├── components/          # Componentes StencilJS
│   └── [component]/
│       ├── [component].tsx      # Lógica do componente
│       ├── [component].types.ts # Definições de tipos
│       ├── [component].scss      # Estilos (barrel file)
│       ├── [component].spec.tsx # Testes unitários
│       ├── [component].stories.ts # Storybook
│       ├── readme.md            # Documentação
│       └── styles/              # Arquivos SCSS (opcional)
│           ├── __base.scss
│           ├── _variant-*.scss
│           └── index.scss
├── shared/
│   ├── theme/
│   │   ├── core/               # Funções e mixins
│   │   └── tokens/             # Design tokens
│   │       ├── actions/        # Tokens de ações (botões)
│   │       ├── feedbacks/      # Tokens de feedback
│   │       ├── inputs/         # Tokens de inputs
│   │       ├── _primitives.scss
│   │       └── _sizing.scss
│   └── assets/fonts/            # Fontes
└── utils/                      # Funções utilitárias
```

## Convenções de Nomenclatura

### Arquivos

- **Componentes**: `kebab-case` (ex: `field-number.tsx`)
- **Tipos**: `[component].types.ts`
- **Estilos**: `[component].scss` ou `styles/__base.scss`
- **Testes**: `[component].spec.tsx`
- **Stories**: `[component].stories.ts`

### Classes CSS

- **BEM-like**: `.mnt-[component]-[element]-[modifier]`
- Exemplo: `.mnt-button-icon-regular`
- Prefixo: `mnt-` (Mantra)

### Variáveis SCSS

- **Componentes**: `$component-prefix: 'mnt-[component]';`
- **Tokens**: `$[token-name]-map: ( ... );`
- **Configurações**: `$[size/variant]-configs: ( ... );`

### TypeScript

- **Classes**: `PascalCase` (ex: `export class Button`)
- **Props**: `camelCase` (ex: `@Prop() iconLeft`)
- **Interfaces**: `PascalCase` + `Props` (ex: `ButtonProps`)
- **Tipos**: `PascalCase` (ex: `IconSize`)

## Estrutura de Componentes StencilJS

### Arquivos Obrigatórios

1. **`[component].tsx`** - Componente principal

```ts
   @Component({
     tag: 'mnt-[component]',
     styleUrl: '[component].scss',
     shadow: false,
   })
   export class ComponentName {
     @Prop() propName: type;

     render() {
       return (
         // JSX
       );
     }
   }
   ```

2. **`[component].types.ts`** - Definições de tipos
   ```typescript
   export const componentPrefix = 'component';

   export interface ComponentBaseProps {
     size?: 'small' | 'medium' | 'large';
     color?: string;
   }

   export interface ComponentProps extends ComponentBaseProps {
     // props específicas
   }
   ```

3. **`[component].scss`** - Estilos (barrel file)
   ```scss
   /**
    * This file is a barrel file for the styles of <mnt-component>.
    * WARNING: Do not add CSS rules directly here.
    * Use the variant files in styles/ directory.
    **/

   @use './styles';
   ```

### Arquivos Opcionais

4. **`styles/`** - Estrutura modular de estilos
   - `__base.scss` - Variáveis e estilos base
   - `_variant-*.scss` - Variantes específicas
   - `index.scss` - Barrel file que importa todos

5. **`[component].stories.ts`** - Documentação Storybook
   ```typescript
   const meta: Meta<ComponentProps> = {
     title: 'Category/Component',
     component: 'mnt-component',
     argTypes: { ... },
   };

   export default meta;
   ```

6. **`[component].spec.tsx`** - Testes unitários

7. **`readme.md`** - Documentação (gerada automaticamente)

## Organização de Estilos (SCSS)

### Structure Pattern (Badge, Button, Field-Number)

Quando um componente tem **múltiplas variantes ou tamanhos**:

```scss
// styles/index.scss (barrel)
@forward './__base';
@use './_variant-plain';
@use './_variant-default';
@use './_variant-simple';

// styles/__base.scss (variáveis e base)
@use 'sass:map';
@use '../../../shared/theme/core/theme' as theme;

$component-prefix: theme.get-prefix('component');

// Variáveis de esquema de cores
$component-color-scheme: (
  default: (...),
  hover: (...),
  focus: (...),
);

// Base styles
mnt-component {
  // base
}

// styles/_variant-[name].scss (variantes específicas)
@use '../../../shared/theme/tokens/...' as tokens;
@use '__base' as base;

$component-size-configs: tokens.$component-size-configs;

mnt-component {
  // Estilos específicos da variante
}
```

### Tokens Centralizados

**Localização**: `src/shared/theme/tokens/`

#### Criar novo token

1. Criar arquivo: `tokens/[category]/_[name].scss`
2. Definir mapa de configurações:
   ```scss
   $component-size-configs: (
     small: (...),
     medium: (...),
     large: (...),
   );
   ```
3. Exportar: `tokens/[category]/index.scss`
   ```scss
   @forward './_[name]';
   ```
4. Reusar em variantes: `@use '../../../shared/theme/tokens/[category]/_[name]' as tokens;`

### Importação de Tokens

```scss
// Usando tokens centralizados
@use '../../../shared/theme/tokens/[category]/_[token]' as token;

$my-configs: token.$my-config-map;

// Acessando valores
$value: map.get(map.get($my-configs, small), property);
```

## TypeScript - Tipos e Interfaces

### Prefixos de Componente

```typescript
export const componentPrefix = 'button';
export const COMPONENT_PREFIX = 'mnt-button';
```

### Props

```typescript
export interface ComponentBaseProps {
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

export interface ComponentProps extends ComponentBaseProps {
  // props específicas
  label?: string;
}
```

### Enums e Arrays

```typescript
export const variantArray = ['variant1', 'variant2'] as const;
export type VariantType = (typeof variantArray)[number];
```

## StencilJS - Lifecycle e Métodos

### Lifecycle Hooks

- `componentWillLoad()` - Antes do primeiro render
- `componentDidLoad()` - Após o primeiro render
- `componentDidUpdate()` - Quando props mudam
- `@Watch('prop')` - Observar mudanças em props

### Renderização

- Usar JSX com TSX
- Sempre retornar JSX.Element
- Usar refs para referenciar elementos DOM

### Eventos

```typescript
@Event() customEvent: EventEmitter<DataType>;
emitCustomEvent(data: DataType) {
  this.customEvent.emit(data);
}
```

## Storybook

### Estrutura de Story

```typescript
const meta: Meta<Props> = {
  title: 'Category/Component',
  component: 'mnt-component',
  argTypes: {
    prop: {
      control: 'select',
      options: [...]
    },
  },
};

const DefaultTemplate = (args: Props) => `...`;
export const Default: StoryFn = DefaultTemplate.bind({});
Default.args = { ... };
```

### Convenções

- Uma story por variante
- Usar nomes de ícones válidos (existentes em icon-base)
- Incluir exemplos de diferentes estados

## Testes

### Estrutura

```typescript
describe('Component', () => {
  it('should render', async () => {
    const { root } = await newSpecPage({
      components: [Component],
      html: `<mnt-component></mnt-component>`,
    });
    expect(root).toEqualHtml(`...`);
  });
});
```

## Checklist de Criação de Componente

- [ ] Criar `[component].tsx` com decoradores adequados
- [ ] Criar `[component].types.ts` com interfaces
- [ ] Criar `[component].scss` (barrel ou direto)
- [ ] Se tiver variantes: criar `styles/` directory
- [ ] Criar `[component].stories.ts` com exemplos
- [ ] Criar `[component].spec.tsx` com testes básicos
- [ ] Se necessário: criar tokens em `shared/theme/tokens/`
- [ ] Atualizar `readme.md` (ou usar auto-gerado)
- [ ] Adicionar ao `index.ts` de exportação
- [ ] Testar no Storybook
- [ ] Executar `npm run build`

## Anti-padrões (NUNCA fazer)

- ❌ Adicionar estilos inline (usar SCSS)
- ❌ Usar IDs em componentes reutilizáveis (usar classes)
- ❌ Duplicar lógica entre componentes (extrair em utils)
- ❌ Hardcode valores (usar tokens)
- ❌ Comentários desnecessários no código
- ❌ Deixar TODOs ou placeholders
- ❌ Usar ícones que não existem em icon-base
- ❌ Criar componentes sem variantes quando faz sentido ter

## Comandos Importantes

```bash
npm run build          # Build de produção
npm run dev            # Dev com watch + Storybook
npm run storybook      # Storybook standalone
npm run test           # Testes unitários
npm run test:e2e       # Testes E2E
npm run generate       # Gerar novo componente
```

## Arquivos de Referência

- **Configuração**: `stencil.config.ts`
- **Tokens globais**: `src/shared/theme/tokens/`
- **Estilos globais**: `src/_common-variables.scss`
- **Exports**: `src/index.ts`
- **Exemplos**: `src/index.html`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yooga-tecnologia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
