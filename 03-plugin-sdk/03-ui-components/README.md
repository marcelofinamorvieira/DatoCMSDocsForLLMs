# UI Components

The `datocms-react-ui` package provides React components that match DatoCMS's design system, ensuring your plugins integrate seamlessly with the interface.

## Installation

```bash
npm install datocms-react-ui react react-dom
```

## Component Categories

### 🎨 Layout Components
Structure and organize your plugin interface:

- **[Canvas](./Canvas.md)** - Main container that applies DatoCMS styling
- **[Section](./Section.md)** - Content sections with titles
- **[SidebarPanel](./SidebarPanel.md)** - Collapsible sidebar sections
- **[SplitView](./SplitView.md)** - Resizable split panels
- **[VerticalSplit](./VerticalSplit.md)** - Vertical content split

### 📝 Form Components
Build forms that match DatoCMS styling:

#### Input Fields
- **[TextField](./TextField.md)** - Single-line text input
- **[TextareaField](./TextareaField.md)** - Multi-line text input
- **[SelectField](./SelectField.md)** - Dropdown selection
- **[SwitchField](./SwitchField.md)** - Toggle switch

#### Form Structure
- **[Form](./Form.md)** - Form container
- **[FormLabel](./FormLabel.md)** - Field labels
- **[FieldGroup](./FieldGroup.md)** - Group related fields
- **[FieldWrapper](./FieldWrapper.md)** - Field container with spacing
- **[FieldHint](./FieldHint.md)** - Help text for fields
- **[FieldError](./FieldError.md)** - Error messages

#### Buttons
- **[Button](./Button.md)** - Standard button with variants
- **[ButtonGroup](./ButtonGroup.md)** - Group multiple buttons

### 🎛 Interactive Components
Advanced UI elements:

- **[Dropdown](./Dropdown.md)** - Context menus and dropdowns
- **[Spinner](./Spinner.md)** - Loading indicator
- **[ContextInspector](./ContextInspector.md)** - Debug panel for development

### 🔧 Toolbar Components
Build custom toolbars:

- **[Toolbar](./Toolbar.md)** - Toolbar container
- **[ToolbarButton](./ToolbarButton.md)** - Toolbar action buttons
- **[ToolbarStack](./ToolbarStack.md)** - Group toolbar items
- **[ToolbarTitle](./ToolbarTitle.md)** - Toolbar section titles

## Basic Usage

### Wrapping Your Plugin

Always wrap your plugin content in a Canvas component:

```jsx
import { Canvas, TextField, Button } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <TextField
        label="Title"
        value={value}
        onChange={setValue}
      />
      <Button onClick={handleSave}>
        Save
      </Button>
    </Canvas>
  );
}
```

### Form Example

Build forms using the form components:

```jsx
import { 
  Canvas, 
  Form, 
  FieldGroup,
  TextField, 
  TextareaField,
  SelectField,
  Button 
} from 'datocms-react-ui';

function MyForm({ ctx }) {
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    category: ''
  });

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <FieldGroup>
          <TextField
            label="Title"
            required
            value={formData.title}
            onChange={(value) => setFormData({...formData, title: value})}
            hint="Enter a descriptive title"
          />
          
          <TextareaField
            label="Description"
            value={formData.description}
            onChange={(value) => setFormData({...formData, description: value})}
            rows={4}
          />
          
          <SelectField
            label="Category"
            value={formData.category}
            onChange={(value) => setFormData({...formData, category: value})}
            options={[
              { label: 'News', value: 'news' },
              { label: 'Blog', value: 'blog' },
              { label: 'Tutorial', value: 'tutorial' }
            ]}
          />
        </FieldGroup>
        
        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </Form>
    </Canvas>
  );
}
```

### Layout Example

Create complex layouts:

```jsx
import { Canvas, SplitView, Section, SidebarPanel } from 'datocms-react-ui';

function MyLayout({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <SplitView>
        <SplitView.Pane width="30%">
          <Section title="Settings">
            <SidebarPanel title="General" startOpen>
              {/* Settings content */}
            </SidebarPanel>
            <SidebarPanel title="Advanced">
              {/* Advanced settings */}
            </SidebarPanel>
          </Section>
        </SplitView.Pane>
        
        <SplitView.Pane>
          <Section title="Preview">
            {/* Main content */}
          </Section>
        </SplitView.Pane>
      </SplitView>
    </Canvas>
  );
}
```

## Styling and Theming

Components automatically adapt to the user's theme (light/dark). The Canvas component applies:

- Consistent typography
- Color schemes
- Spacing and layout rules
- Responsive behavior

### Custom Styling

While components come pre-styled, you can add custom styles:

```jsx
<div style={{ marginTop: 'var(--spacing-m)' }}>
  <Button>Custom Spacing</Button>
</div>
```

### CSS Variables

Access DatoCMS design tokens:

```css
/* Spacing */
--spacing-xs: 4px
--spacing-s: 8px
--spacing-m: 16px
--spacing-l: 24px
--spacing-xl: 32px

/* Colors */
--primary-color
--accent-color
--background-color
--text-color
--border-color

/* Typography */
--font-size-xs
--font-size-s
--font-size-m
--font-size-l
```

## Accessibility

All components follow accessibility best practices:

- Proper ARIA labels
- Keyboard navigation
- Focus management
- Screen reader support

## TypeScript Support

All components are fully typed:

```typescript
import { TextFieldProps, ButtonProps } from 'datocms-react-ui';

const customProps: TextFieldProps = {
  label: 'Custom Field',
  value: '',
  onChange: (value: string) => console.log(value),
  required: true,
  error: 'Field is required'
};
```

## Best Practices

1. **Always use Canvas**: Wrap your plugin root in Canvas for proper styling
2. **Consistent spacing**: Use FieldGroup and Section for proper layout
3. **Loading states**: Show Spinner during async operations
4. **Error handling**: Use FieldError to display validation errors
5. **Responsive design**: Components are responsive by default
6. **Theme support**: Don't hardcode colors, use CSS variables

## Component Composition

Build complex UIs by composing components:

```jsx
function ComplexField({ ctx, value, onChange }) {
  return (
    <Canvas ctx={ctx}>
      <FieldWrapper>
        <FormLabel required>Complex Field</FormLabel>
        <div style={{ display: 'flex', gap: 'var(--spacing-s)' }}>
          <TextField
            value={value.part1}
            onChange={(v) => onChange({ ...value, part1: v })}
            placeholder="Part 1"
          />
          <TextField
            value={value.part2}
            onChange={(v) => onChange({ ...value, part2: v })}
            placeholder="Part 2"
          />
        </div>
        <FieldHint>Enter both parts of the complex field</FieldHint>
        {error && <FieldError>{error}</FieldError>}
      </FieldWrapper>
    </Canvas>
  );
}
```

## Related Documentation

- [Plugin SDK Overview](../README.md)
- [Render Field Extension Hook](../02-hooks/renderFieldExtension.md)
- [Context Object](../01-core-concepts/context-object.md)