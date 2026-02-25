# MUI vs Tailwind CSS Analysis — Task Rabbit

## Summary

MUI is the dominant styling framework. Tailwind is used in only 4 files. **Recommendation: keep MUI, remove Tailwind.**

## Usage Comparison

| Metric              | MUI        | Tailwind   |
|---------------------|------------|------------|
| Files using it      | 75         | 4          |
| Import/usage count  | 90         | ~133       |
| % of source files   | 62.5%      | 3.3%       |

## MUI Usage

- **75 files** import from `@mui/material` or `@mui/icons-material`
- All 31 UI components in `src/components/ui/` use MUI
- 13+ page components use MUI
- Custom theme defined in `src/theme/index.ts` with design tokens
- Styling pattern: `styled()` from `@mui/material` wrapping MUI components

### Dependencies

- `@mui/material: ^7.3.4`
- `@mui/icons-material: ^7.3.4`
- `@emotion/react: ^11.14.0`
- `@emotion/styled: ^11.14.1`

## Tailwind Usage

Only **4 files** use Tailwind utility classes:

1. `src/components/ui/Footer.tsx`
2. `src/components/ui/Header.tsx`
3. `src/components/ui/SearchBar.tsx`
4. `src/app/(pages)/booktask/page.tsx`

Most common utilities: `text-*`, `flex`, `w-*`, `gap-*`, `bg-*`, `px-*`, `py-*`, `rounded`.

### Dependencies

- `tailwindcss: ^3.3.0`
- `autoprefixer: ^10.0.1`
- `postcss: ^8`

## Recommendation

**Keep MUI, remove Tailwind.**

- MUI is the established design system with a custom theme and tokens
- Only 4 files need migration (convert Tailwind classes to MUI `styled()` or `sx` props)
- Eliminates Tailwind config, PostCSS setup, and cognitive overhead of two systems
- Aligns the entire codebase to one consistent styling approach

## Migration Steps

1. Convert `Header.tsx`, `Footer.tsx`, `SearchBar.tsx`, and `booktask/page.tsx` to MUI styling
2. Remove `tailwindcss`, `autoprefixer`, `postcss` dependencies
3. Delete `tailwind.config.js` and `postcss.config.js`
4. Remove Tailwind directives from global CSS

---

## Slack formatted

*MUI vs Tailwind CSS Analysis — Task Rabbit*

*Summary:* MUI is the dominant styling framework. Tailwind is used in only 4 files. *Recommendation: keep MUI, remove Tailwind.*

*Usage Comparison*
• Files using it — MUI: *75* | Tailwind: *4*
• Import/usage count — MUI: *90* | Tailwind: *~133*
• % of source files — MUI: *62.5%* | Tailwind: *3.3%*

*MUI Usage*
• 75 files import from `@mui/material` or `@mui/icons-material`
• All 31 UI components in `src/components/ui/` use MUI
• 13+ page components use MUI
• Custom theme in `src/theme/index.ts` with design tokens
• Styling pattern: `styled()` wrapping MUI components

*Tailwind Usage*
Only 4 files use Tailwind:
1. `src/components/ui/Footer.tsx`
2. `src/components/ui/Header.tsx`
3. `src/components/ui/SearchBar.tsx`
4. `src/app/(pages)/booktask/page.tsx`

*Recommendation: Keep MUI, remove Tailwind*
• MUI is the established design system with a custom theme and tokens
• Only 4 files need migration
• Eliminates Tailwind config, PostCSS setup, and cognitive overhead of two systems

*Migration Steps*
1. Convert the 4 Tailwind files to MUI `styled()` / `sx` props
2. Remove `tailwindcss`, `autoprefixer`, `postcss` deps
3. Delete `tailwind.config.js` and `postcss.config.js`
4. Remove Tailwind directives from global CSS
