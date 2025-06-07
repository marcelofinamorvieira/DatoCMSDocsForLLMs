# UI Components

The DatoCMS React UI library (`datocms-react-ui`) provides a comprehensive set of React components that match the DatoCMS interface design system. These components help you build plugin UIs that look and feel native to the DatoCMS environment.

## Installation

```bash
npm install datocms-react-ui
```

## Core Components

### Layout Components

- [Canvas](./Canvas.md) - **Required root component** that provides theming and context
- [Form](./Form.md) - Form container with consistent spacing
- [Section](./Section.md) - Collapsible content sections
- [SidebarPanel](./SidebarPanel.md) - Collapsible panels for sidebars
- [SplitView](./SplitView.md) - Advanced resizable split pane layouts
- [VerticalSplit](./VerticalSplit.md) - Simple two-pane vertical split
- [Toolbar](./Toolbar.md) - Toolbar system with title and actions

### Form Components

#### Complete Fields (with labels and error handling)
- [TextField](./TextField.md) - Single-line text field with label
- [TextareaField](./TextareaField.md) - Multi-line text field with label
- [SelectField](./SelectField.md) - Dropdown field with label
- [SwitchField](./SwitchField.md) - Toggle switch with label

#### Input Components (standalone)
- [TextInput](./TextInput.md) - Single-line text input
- [TextareaInput](./TextareaInput.md) - Multi-line text input
- [SelectInput](./SelectInput.md) - Dropdown select input
- [SwitchInput](./SwitchInput.md) - Toggle switch input

#### Form Utilities
- [FieldWrapper](./FieldWrapper.md) - Container for custom form fields
- [FieldGroup](./FieldGroup.md) - Groups related form fields
- [FormLabel](./FormLabel.md) - Accessible label component
- [FieldHint](./FieldHint.md) - Help text for form fields
- [FieldError](./FieldError.md) - Error message display

### Interactive Components

- [Button](./Button.md) - Primary interactive element
- [ButtonGroup](./ButtonGroup.md) - Group related buttons
- [Dropdown](./Dropdown.md) - Flexible dropdown menu system
- [Spinner](./Spinner.md) - Loading indicator

### Utility Components & Hooks

- [ContextInspector](./ContextInspector.md) - Debug plugin context
- [useClickOutside](./useClickOutside.md) - Detect clicks outside element
- [useMediaQuery](./useMediaQuery.md) - Responsive design utilities
- [icons](./icons.md) - Built-in SVG icons

## Basic Setup

Every DatoCMS plugin UI must wrap its components in the Canvas component:

```jsx
import { Canvas, Button } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <h1>My Plugin</h1>
      <Button onClick={() => console.log('Clicked!')}>
        Click me
      </Button>
    </Canvas>
  );
}
```

## Common Patterns

### Form Layout

```jsx
import { 
  Canvas, 
  Form, 
  TextField, 
  TextareaField, 
  SelectField,
  SwitchField,
  Button 
} from 'datocms-react-ui';

function SettingsForm({ ctx }) {
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    category: '',
    isPublic: false
  });

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <TextField
          label="Title"
          value={formData.title}
          onChange={(value) => setFormData({ ...formData, title: value })}
          required
        />
        
        <TextareaField
          label="Description"
          value={formData.description}
          onChange={(value) => setFormData({ ...formData, description: value })}
          rows={4}
        />
        
        <SelectField
          label="Category"
          value={formData.category}
          onChange={(value) => setFormData({ ...formData, category: value })}
        >
          <option value="">Select a category</option>
          <option value="news">News</option>
          <option value="blog">Blog</option>
        </SelectField>
        
        <SwitchField
          label="Make public"
          value={formData.isPublic}
          onChange={(value) => setFormData({ ...formData, isPublic: value })}
        />
        
        <Button type="submit" buttonType="primary">
          Save Settings
        </Button>
      </Form>
    </Canvas>
  );
}
```

### Split Layout with Toolbar

```jsx
import { 
  Canvas, 
  Toolbar, 
  VerticalSplit,
  Button 
} from 'datocms-react-ui';

function EditorLayout({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar.Toolbar>
        <Toolbar.Stack>
          <Toolbar.Title>Content Editor</Toolbar.Title>
        </Toolbar.Stack>
        <Toolbar.Stack>
          <Toolbar.Button onClick={handleSave}>Save</Toolbar.Button>
          <Toolbar.Button onClick={handleCancel}>Cancel</Toolbar.Button>
        </Toolbar.Stack>
      </Toolbar.Toolbar>
      
      <VerticalSplit split={30}>
        <div>Sidebar content</div>
        <div>Main content</div>
      </VerticalSplit>
    </Canvas>
  );
}
```

### Responsive Design

```jsx
import { Canvas, useMediaQuery } from 'datocms-react-ui';

function ResponsivePlugin({ ctx }) {
  const isMobile = useMediaQuery('(max-width: 768px)');

  return (
    <Canvas ctx={ctx}>
      <div style={{
        display: 'grid',
        gridTemplateColumns: isMobile ? '1fr' : 'repeat(3, 1fr)',
        gap: 'var(--spacing-m)'
      }}>
        {/* Content adapts to screen size */}
      </div>
    </Canvas>
  );
}
```

## Styling

All components use CSS custom properties (CSS variables) that automatically adapt to the DatoCMS theme:

- `--primary-color` - Primary brand color
- `--light-color` - Light background color
- `--border-color` - Border color
- `--text-color` - Main text color
- `--muted-color` - Muted text color
- `--alert-color` - Error/alert color
- `--success-color` - Success color
- `--spacing-s/m/l/xl` - Consistent spacing values
- `--border-radius-s/m/l` - Border radius values
- `--font-size-s/m/l` - Font size values

## Best Practices

1. **Always use Canvas**: Wrap your entire plugin UI in the Canvas component
2. **Consistent spacing**: Use the spacing CSS variables for consistent layouts
3. **Responsive design**: Use useMediaQuery for adaptive layouts
4. **Accessibility**: Use semantic HTML and proper ARIA attributes
5. **Form validation**: Provide clear error messages and field hints
6. **Loading states**: Show Spinner during async operations
7. **Keyboard navigation**: Ensure all interactive elements are keyboard accessible

## Component Categories

### Essential Setup
Start with these components:
- Canvas (required wrapper)
- Button (primary interaction)
- Form + TextField (basic input)

### Layout Building
For complex layouts:
- Toolbar (app-like header)
- VerticalSplit or SplitView (multi-pane layouts)
- Section (organized content)

### Form Building
For data input:
- Use complete field components (TextField, TextareaField, etc.) for standard forms
- Use input components (TextInput, TextareaInput, etc.) for custom layouts
- Combine with FieldWrapper for custom field types

### Responsive Design
For adaptive UIs:
- useMediaQuery hook for breakpoints
- Conditional rendering based on screen size
- Flexible grid/flexbox layouts

## TypeScript Support

All components are fully typed. Import types as needed:

```typescript
import { ButtonProps, Canvas } from 'datocms-react-ui';
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

function MyPlugin({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  return (
    <Canvas ctx={ctx}>
      {/* Your plugin UI */}
    </Canvas>
  );
}
```

## Next Steps

1. Start with the [Canvas](./Canvas.md) component documentation
2. Explore form components for data input
3. Learn about layout components for complex UIs
4. Check utility hooks for advanced functionality