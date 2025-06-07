# Button Component

The Button component provides a consistent way to render buttons throughout your plugin interface. It supports different types, sizes, icons, and states to match DatoCMS's design system. The package also exports a ButtonLink component for link-styled buttons.

## Import

```jsx
import { Button, ButtonLink } from 'datocms-react-ui';
```

## Components

### Button

The main button component for interactive actions.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Button content |
| `type` | `'button' \| 'submit' \| 'reset'` | `'button'` | HTML button type |
| `className` | `string` | - | Additional CSS classes |
| `disabled` | `boolean` | `false` | Whether button is disabled |
| `onClick` | `MouseEventHandler` | - | Click handler |
| `buttonType` | `'primary' \| 'muted' \| 'negative'` | `'muted'` | Visual style variant |
| `buttonSize` | `'xxs' \| 'xs' \| 's' \| 'm' \| 'l' \| 'xl'` | `'m'` | Button size |
| `leftIcon` | `ReactNode` | - | Icon to show on the left |
| `rightIcon` | `ReactNode` | - | Icon to show on the right |
| `fullWidth` | `boolean` | `false` | Whether button takes full width |
| `style` | `CSSProperties` | - | Inline styles |

### ButtonLink

A link styled as a button, for navigation purposes.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Button content |
| `href` | `string` | (required) | Link destination URL |
| `target` | `string` | `'_blank'` | Link target attribute |
| `className` | `string` | - | Additional CSS classes |
| `onClick` | `MouseEventHandler` | - | Click handler |
| `buttonType` | `'primary' \| 'muted' \| 'negative'` | `'muted'` | Visual style variant |
| `buttonSize` | `'xxs' \| 'xs' \| 's' \| 'm' \| 'l' \| 'xl'` | `'m'` | Button size |
| `leftIcon` | `ReactNode` | - | Icon to show on the left |
| `rightIcon` | `ReactNode` | - | Icon to show on the right |
| `fullWidth` | `boolean` | `false` | Whether button takes full width |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Button, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Button onClick={() => console.log('Clicked!')}>
        Click me
      </Button>
    </Canvas>
  );
}
```

## Button Types

### Primary Button

Used for main actions:

```jsx
<Button buttonType="primary" onClick={handleSave}>
  Save Changes
</Button>
```

### Negative Button

Used for destructive actions:

```jsx
<Button buttonType="negative" onClick={handleDelete}>
  Delete Item
</Button>
```

### Muted Button (Default)

Used for secondary actions:

```jsx
<Button buttonType="muted" onClick={handleCancel}>
  Cancel
</Button>
```

## Sizes

All available sizes from smallest to largest:

```jsx
<Button buttonSize="xxs">Extra Extra Small</Button>
<Button buttonSize="xs">Extra Small</Button>
<Button buttonSize="s">Small</Button>
<Button buttonSize="m">Medium (default)</Button>
<Button buttonSize="l">Large</Button>
<Button buttonSize="xl">Extra Large</Button>
```

## With Icons

```jsx
import { FaPlus, FaArrowRight } from 'react-icons/fa';

// Left icon
<Button leftIcon={<FaPlus />}>
  Add Item
</Button>

// Right icon
<Button rightIcon={<FaArrowRight />}>
  Continue
</Button>

// Both icons
<Button leftIcon={<FaPlus />} rightIcon={<FaArrowRight />}>
  Add and Continue
</Button>

// Icon-only button
<Button leftIcon={<FaPlus />} />
```

## States

### Disabled State

```jsx
<Button disabled>
  Not Available
</Button>

<Button buttonType="primary" disabled>
  Disabled Primary
</Button>

<Button buttonType="negative" disabled>
  Disabled Negative
</Button>
```

### Full Width

```jsx
<Button fullWidth buttonType="primary">
  Full Width Button
</Button>
```

## ButtonLink Usage

For navigation that looks like a button:

```jsx
<ButtonLink href="https://www.datocms.com" target="_blank">
  Visit DatoCMS
</ButtonLink>

<ButtonLink 
  href="/settings" 
  target="_self"
  buttonType="primary"
  leftIcon={<SettingsIcon />}
>
  Open Settings
</ButtonLink>
```

## Form Integration

```jsx
import { Form, Button } from 'datocms-react-ui';

function MyForm({ ctx }) {
  const handleSubmit = (e) => {
    e.preventDefault();
    // Handle form submission
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        {/* Form fields */}
        <Button type="submit" buttonType="primary">
          Submit Form
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Advanced Examples

### Async Action

```jsx
function SaveButton({ ctx, data }) {
  const [saving, setSaving] = useState(false);

  const handleSave = async () => {
    setSaving(true);
    try {
      await ctx.updatePluginParameters(data);
      ctx.notice('Saved successfully!');
    } catch (error) {
      ctx.alert('Failed to save: ' + error.message);
    } finally {
      setSaving(false);
    }
  };

  return (
    <Button 
      buttonType="primary" 
      disabled={saving}
      onClick={handleSave}
    >
      {saving ? 'Saving...' : 'Save'}
    </Button>
  );
}
```

### Confirmation Button

```jsx
function DeleteButton({ ctx, itemId }) {
  const handleDelete = async () => {
    const confirmed = await ctx.openConfirm({
      title: 'Delete Item',
      content: 'Are you sure you want to delete this item?',
      cancel: { label: 'Cancel', value: false },
      choices: [
        { label: 'Delete', value: true, intent: 'negative' }
      ]
    });

    if (confirmed) {
      // Perform deletion
    }
  };

  return (
    <Button buttonType="negative" onClick={handleDelete}>
      Delete
    </Button>
  );
}
```

### Dynamic Button

```jsx
function DynamicButton({ ctx, status }) {
  const getButtonProps = () => {
    switch (status) {
      case 'draft':
        return {
          buttonType: 'primary',
          children: 'Publish',
          onClick: handlePublish
        };
      case 'published':
        return {
          buttonType: 'muted',
          children: 'Unpublish',
          onClick: handleUnpublish
        };
      default:
        return {
          disabled: true,
          children: 'Unknown Status'
        };
    }
  };

  return <Button {...getButtonProps()} />;
}
```

## Styling

### Custom Styling

```jsx
<Button 
  style={{ 
    marginTop: 'var(--spacing-m)',
    minWidth: '120px' 
  }}
  className="my-custom-button"
>
  Custom Styled
</Button>
```

## TypeScript

```typescript
import { Button, ButtonLink, ButtonProps, ButtonLinkProps } from 'datocms-react-ui';

// Button
const buttonProps: ButtonProps = {
  buttonType: 'primary',
  buttonSize: 'm',
  onClick: () => console.log('clicked'),
  children: 'Click me',
  disabled: false
};

<Button {...buttonProps} />

// ButtonLink
const linkProps: ButtonLinkProps = {
  href: 'https://example.com',
  target: '_blank',
  buttonType: 'primary',
  children: 'Visit Site'
};

<ButtonLink {...linkProps} />
```

## Type Definitions

```typescript
export type ButtonProps = {
  children?: ReactNode;
  type?: React.ButtonHTMLAttributes<HTMLButtonElement>['type'];
  className?: string;
  disabled?: boolean;
  onClick?: MouseEventHandler;
  buttonType?: 'primary' | 'muted' | 'negative';
  buttonSize?: 'xxs' | 'xs' | 's' | 'm' | 'l' | 'xl';
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
  fullWidth?: boolean;
  style?: CSSProperties;
};

export type ButtonLinkProps = {
  children?: ReactNode;
  href: string;
  target?: React.AnchorHTMLAttributes<HTMLAnchorElement>['target'];
  className?: string;
  onClick?: MouseEventHandler;
  buttonType?: 'primary' | 'muted' | 'negative';
  buttonSize?: 'xxs' | 'xs' | 's' | 'm' | 'l' | 'xl';
  fullWidth?: boolean;
  style?: CSSProperties;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
};
```

## Best Practices

1. **Use appropriate variants**: Primary for main actions, negative for destructive
2. **Provide feedback**: Show loading or disabled states during async operations
3. **Disable during operations**: Prevent double-clicks with `disabled`
4. **Clear labels**: Use descriptive text that explains the action
5. **Icon usage**: Use icons to enhance, not replace, text labels
6. **Confirmation for destructive actions**: Always confirm before deleting
7. **Use ButtonLink for navigation**: Don't use Button with onClick for navigation

## Related Components

- [ButtonGroup](./ButtonGroup.md) - Group multiple buttons
- [Form](./Form.md) - Form container
- [Toolbar](./Toolbar.md) - Toolbar with buttons