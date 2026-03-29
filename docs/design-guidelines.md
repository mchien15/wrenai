# WrenAI - Design Guidelines

## Overview

This document outlines the UI/UX design patterns, component usage, and styling standards used in WrenAI. It serves as a reference for maintaining design consistency across the application.

## Design Philosophy

WrenAI follows these core design principles:

1. **Clarity Over Complexity**: Clean, intuitive interfaces that reduce cognitive load
2. **Data-First Design**: Information hierarchy centered on data visualization
3. **Progressive Disclosure**: Show essential info first, reveal details on demand
4. **Responsive by Default**: Mobile-first approach with breakpoints for all screen sizes
5. **Accessibility First**: WCAG 2.1 AA compliance with keyboard navigation support

## Color System

### Primary Colors

```typescript
// Light mode
const colors = {
  primary: '#1677ff',        // Brand blue (Ant Design default)
  primaryHover: '#4096ff',
  primaryActive: '#0958d9',
  primaryBg: '#e6f4ff',

  success: '#52c41a',        // Green for success states
  warning: '#faad14',        // Yellow for warnings
  error: '#ff4d4f',          // Red for errors
  info: '#1677ff',           // Blue for info
};
```

### Semantic Colors

```typescript
const semanticColors = {
  textPrimary: 'rgba(0, 0, 0, 0.88)',      // Main text
  textSecondary: 'rgba(0, 0, 0, 0.65)',    // Secondary text
  textTertiary: 'rgba(0, 0, 0, 0.45)',     // Disabled/hint text
  border: '#d9d9d9',                       // Default border
  divider: 'rgba(5, 5, 5, 0.06)',         // Divider lines
  background: '#ffffff',                   // Default bg
  backgroundLayout: '#f5f5f5',             // Layout bg
};
```

### Dark Mode

```typescript
const darkColors = {
  primary: '#177ddc',
  textPrimary: 'rgba(255, 255, 255, 0.85)',
  textSecondary: 'rgba(255, 255, 255, 0.65)',
  background: '#141414',
  backgroundLayout: '#000000',
  border: '#424242',
};
```

## Typography

### Font Families

```typescript
const typography = {
  fontFamily: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
               'Helvetica Neue', Arial, sans-serif`,
  monoFont: `'SFMono-Regular', Consolas, 'Liberation Mono', Menlo,
              Courier, monospace`,
};
```

### Type Scale

```typescript
const fontSize = {
  fontSizeSM: '12px',      // Small text, captions
  fontSize: '14px',        // Default body text
  fontSizeLG: '16px',      // Large text
  fontSizeXL: '20px',      // Extra large
  fontSizeHeading1: '38px', // H1
  fontSizeHeading2: '30px', // H2
  fontSizeHeading3: '24px', // H3
  fontSizeHeading4: '20px', // H4
  fontSizeHeading5: '16px', // H5
};

const lineHeight = {
  lineHeight: 1.5715,      // Default line height
  lineHeightLG: 1.5,       // Large line height
  lineHeightSM: 1.66,      // Small line height
};
```

### Font Weights

```typescript
const fontWeight = {
  fontWeightNormal: 400,
  fontWeightStrong: 500,   // Medium
  fontWeightBold: 600,     // Semibold
};
```

## Spacing System

```typescript
const spacing = {
  XS: '4px', SM: '8px', MD: '16px', LG: '24px', XL: '32px', XXL: '48px',
};
// Component padding: 4px (compact), 8px (small), 16px (default), 24px (large), 32px (xl)
// Layout margins follow the same scale
```

## Component Library: Ant Design

WrenAI uses [Ant Design](https://ant.design/) as the primary component library.

### Core Components

#### Buttons

- `type="primary"` for main CTA (one per page), default for secondary, `danger` for destructive
- Add icons to clarify action purpose

#### Forms

- Use `layout="vertical"` for better mobile experience
- Always provide clear labels, inline validation, group related fields with `Collapse`/`Card`

#### Tables

- Use `rowKey` for efficient updates, pagination for >20 items, sorting for sortable columns

#### Modals

- Clear action-oriented titles, `okText`/`cancelText` for clarity, don't nest modals

#### Notifications

- `message` for brief feedback (3s), `notification` for detailed feedback (4.5s)

## Layout Patterns

### Page Structure

```typescript
import { Layout, Breadcrumb } from 'antd';

const { Header, Content, Sider } = Layout;

<Layout>
  <Header>{/* Logo, nav, user menu */}</Header>
  <Layout>
    <Sider width={256}>
      {/* Navigation menu */}
    </Sider>
    <Content style={{ padding: '24px' }}>
      <Breadcrumb>
        <Breadcrumb.Item>Home</Breadcrumb.Item>
        <Breadcrumb.Item>Projects</Breadcrumb.Item>
      </Breadcrumb>
      <div style={{ marginTop: '24px' }}>
        {/* Page content */}
      </div>
    </Content>
  </Layout>
</Layout>;
```

### Grid System

```typescript
import { Row, Col } from 'antd';

<Row gutter={[16, 16]}>
  <Col xs={24} sm={12} md={8} lg={6}>
    {/* Card component */}
  </Col>
  <Col xs={24} sm={12} md={8} lg={6}>
    {/* Card component */}
  </Col>
</Row>
```

**Breakpoints**:
- `xs`: <576px (mobile)
- `sm`: ≥576px (tablet)
- `md`: ≥768px (small desktop)
- `lg`: ≥992px (desktop)
- `xl`: ≥1200px (large desktop)
- `xxl`: ≥1600px (extra large)

### Dashboard Layouts

Uses `react-grid-layout` with 12-column grid, draggable/resizable widgets for dashboard items.

## Custom Components

### Data Source Cards
Hoverable `Card` with icon, name, and connection status `Tag`.

### SQL Editor
Monospace `TextArea` with auto-sizing, used for SQL input.

### Query Result Table
Ant `Table` with dynamic columns from query metadata, horizontal scroll, small size.

## Data Visualization

### Chart Types

WrenAI uses [Vega](https://vega.github.io/vega/) and [Vega-Lite](https://vega.github.io/vega-lite/) for declarative visualization.

#### Chart Selection Guide

| Data Type | Recommended Chart | Vega Type |
|-----------|-------------------|-----------|
| Comparison (2-3 values) | Bar chart | `bar` |
| Comparison (many values) | Horizontal bar | `bar` (flipped) |
| Trend over time | Line chart | `line` |
| Part-to-whole | Pie chart | `arc` |
| Distribution | Histogram | `bar` (binned) |
| Correlation | Scatter plot | `point` |
| Geospatial | Map | `geoshape` |

#### Example: Bar Chart

```typescript
const barChartSpec = {
  $schema: 'https://vega.github.io/schema/vega-lite/v5.json',
  data: { values: chartData },
  mark: 'bar',
  encoding: {
    x: {
      field: 'category',
      type: 'nominal',
      axis: { labelAngle: -45 },
    },
    y: {
      field: 'value',
      type: 'quantitative',
      title: 'Count',
    },
    color: {
      field: 'category',
      type: 'nominal',
      legend: null,
    },
  },
};
```

#### Chart Component
Renders via `vega-embed` with declarative spec, configurable width/height, actions disabled.

## Interactive Elements

### Model Diagrams (ReactFlow)
`ReactFlow` with `fitView` for rendering data model relationship diagrams with nodes and edges.

### Drag and Drop
`react-dnd` (`DndProvider`, `useDrag`, `useDrop`) for reorderable items.

## Responsive Design

### Mobile Breakpoints

```typescript
const breakpoints = {
  sm: '576px',
  md: '768px',
  lg: '992px',
  xl: '1200px',
  xxl: '1600px',
};

// Usage in styled-components
const ResponsiveContainer = styled.div`
  padding: 16px;

  @media (min-width: ${breakpoints.md}) {
    padding: 24px;
  }

  @media (min-width: ${breakpoints.lg}) {
    padding: 32px;
  }
`;
```

### Mobile Navigation
Ant `Drawer` with `Menu` for mobile sidebar navigation.

### ARIA Labels & Keyboard Navigation
- Add `aria-label` on icon-only buttons, `aria-required` on required inputs
- Enter/Escape key handlers for modals and forms, `tabIndex` for focusable containers

## Styling Patterns

### Styled Components
CSS-in-JS via `styled-components`. Custom styles wrap Ant Design components with hover effects, border-radius, shadows.

### Theme Customization
Ant `ConfigProvider` with `theme.token` (colorPrimary: `#1677ff`, borderRadius: 8, fontSize: 14).

## Icon Usage
Ant Design icons (`@ant-design/icons`) for standard actions. Custom SVG icons via `Icon` component.

## Animation Guidelines

### Transitions
CSS keyframe animations via styled-components (e.g., fadeIn 0.3s ease-in).

### Loading States
Ant `Spin` component with `spinning`, `tip`, `delay` props for async operations.

## Error States

### Error Boundaries
React `ErrorBoundary` class catches errors, renders `Alert` with error message.

### Empty States
Ant `Empty` component with description and CTA button for no-data states.

## Performance Considerations

- **Code Splitting**: `lazy()` + `Suspense` for route-level splitting
- **Memoization**: `memo()` for expensive components

## Design Resources

- **Ant Design**: https://ant.design/
- **Vega/Vega-Lite**: https://vega.github.io/
- **React Flow**: https://reactflow.dev/
- **Styled Components**: https://styled-components.com/

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
