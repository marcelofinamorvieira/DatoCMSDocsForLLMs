# TextareaInput Component

A standalone multi-line text input component that provides just the textarea element without label or error handling, matching DatoCMS's design system.

## Import

```jsx
import { TextareaInput } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | - | Current value |
| `onChange` | `(e: ChangeEvent<HTMLTextAreaElement>) => void` | - | Change handler |
| `placeholder` | `string` | - | Placeholder text |
| `disabled` | `boolean` | `false` | Disable the textarea |
| `readOnly` | `boolean` | `false` | Make read-only |
| `error` | `boolean` | `false` | Show error state |
| `rows` | `number` | `4` | Number of visible rows |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |
| ...rest | `TextareaHTMLAttributes` | - | Standard textarea attributes |

## Basic Usage

```jsx
import { TextareaInput, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [value, setValue] = useState('');

  return (
    <Canvas ctx={ctx}>
      <TextareaInput
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Enter your text here..."
        rows={6}
      />
    </Canvas>
  );
}
```

## Common Patterns

### With Custom Label

```jsx
function CustomLabelTextarea() {
  const [value, setValue] = useState('');

  return (
    <div>
      <label 
        htmlFor="custom-textarea"
        style={{ 
          display: 'block',
          marginBottom: 'var(--spacing-xs)',
          fontWeight: 'bold'
        }}
      >
        Custom Label
      </label>
      <TextareaInput
        id="custom-textarea"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        rows={5}
      />
    </div>
  );
}
```

### With Character Counter

```jsx
function TextareaWithCounter() {
  const [value, setValue] = useState('');
  const maxLength = 500;

  return (
    <div>
      <TextareaInput
        value={value}
        onChange={(e) => setValue(e.target.value)}
        maxLength={maxLength}
        rows={4}
      />
      <div style={{ 
        textAlign: 'right',
        fontSize: 'var(--font-size-s)',
        color: value.length > maxLength * 0.9 ? 'var(--alert-color)' : 'var(--muted-color)',
        marginTop: 'var(--spacing-xs)'
      }}>
        {value.length}/{maxLength}
      </div>
    </div>
  );
}
```

### Auto-Resizing

```jsx
function AutoResizingTextarea() {
  const [value, setValue] = useState('');
  const textareaRef = useRef(null);

  useEffect(() => {
    if (textareaRef.current) {
      textareaRef.current.style.height = 'auto';
      textareaRef.current.style.height = `${textareaRef.current.scrollHeight}px`;
    }
  }, [value]);

  return (
    <TextareaInput
      ref={textareaRef}
      value={value}
      onChange={(e) => setValue(e.target.value)}
      style={{ 
        minHeight: '100px',
        resize: 'none',
        overflow: 'hidden'
      }}
      rows={1}
    />
  );
}
```

## Advanced Examples

### Markdown Preview

```jsx
function MarkdownEditor() {
  const [value, setValue] = useState('');
  const [activeTab, setActiveTab] = useState('edit');

  return (
    <div>
      <div style={{ 
        display: 'flex',
        gap: 'var(--spacing-xs)',
        marginBottom: 'var(--spacing-s)'
      }}>
        <Button
          size="s"
          buttonType={activeTab === 'edit' ? 'primary' : 'muted'}
          onClick={() => setActiveTab('edit')}
        >
          Edit
        </Button>
        <Button
          size="s"
          buttonType={activeTab === 'preview' ? 'primary' : 'muted'}
          onClick={() => setActiveTab('preview')}
        >
          Preview
        </Button>
      </div>
      
      {activeTab === 'edit' ? (
        <TextareaInput
          value={value}
          onChange={(e) => setValue(e.target.value)}
          placeholder="Enter markdown..."
          rows={10}
          style={{ fontFamily: 'monospace' }}
        />
      ) : (
        <div style={{
          minHeight: '250px',
          padding: 'var(--spacing-m)',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          backgroundColor: 'var(--light-color)'
        }}>
          <ReactMarkdown>{value || '*Nothing to preview*'}</ReactMarkdown>
        </div>
      )}
    </div>
  );
}
```

### Code Editor

```jsx
function CodeEditor({ language, onChange }) {
  const [code, setCode] = useState('');
  const [error, setError] = useState(false);

  const handleChange = (e) => {
    const newCode = e.target.value;
    setCode(newCode);
    
    // Basic validation
    try {
      if (language === 'json') {
        JSON.parse(newCode || '{}');
      }
      setError(false);
      onChange(newCode);
    } catch {
      setError(true);
    }
  };

  return (
    <div>
      <div style={{ 
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: 'var(--spacing-xs)'
      }}>
        <span style={{ fontWeight: 'bold' }}>{language.toUpperCase()}</span>
        {error && (
          <span style={{ color: 'var(--alert-color)', fontSize: 'var(--font-size-s)' }}>
            Invalid {language}
          </span>
        )}
      </div>
      
      <TextareaInput
        value={code}
        onChange={handleChange}
        error={error}
        rows={15}
        style={{
          fontFamily: 'monospace',
          fontSize: '13px',
          lineHeight: '1.5',
          tabSize: 2
        }}
        spellCheck={false}
      />
    </div>
  );
}
```

### Note Taking with Timestamps

```jsx
function NotesWithTimestamps() {
  const [notes, setNotes] = useState('');
  const [history, setHistory] = useState([]);

  const addTimestamp = () => {
    const timestamp = new Date().toLocaleString();
    const newEntry = `\n\n--- ${timestamp} ---\n`;
    setNotes(notes + newEntry);
    
    // Focus at the end
    setTimeout(() => {
      const textarea = document.getElementById('notes-textarea');
      if (textarea) {
        textarea.focus();
        textarea.setSelectionRange(textarea.value.length, textarea.value.length);
      }
    }, 0);
  };

  const saveNote = () => {
    if (notes.trim()) {
      setHistory([...history, { content: notes, savedAt: new Date() }]);
      setNotes('');
    }
  };

  return (
    <div>
      <div style={{ marginBottom: 'var(--spacing-s)' }}>
        <Button size="s" onClick={addTimestamp}>
          Add Timestamp
        </Button>
      </div>
      
      <TextareaInput
        id="notes-textarea"
        value={notes}
        onChange={(e) => setNotes(e.target.value)}
        placeholder="Start typing your notes..."
        rows={8}
      />
      
      <Button 
        onClick={saveNote}
        disabled={!notes.trim()}
        style={{ marginTop: 'var(--spacing-s)' }}
      >
        Save Note
      </Button>
      
      {history.length > 0 && (
        <div style={{ marginTop: 'var(--spacing-l)' }}>
          <h4>Previous Notes</h4>
          {history.map((note, index) => (
            <div 
              key={index}
              style={{ 
                marginBottom: 'var(--spacing-m)',
                padding: 'var(--spacing-s)',
                backgroundColor: 'var(--light-color)',
                borderRadius: 'var(--border-radius-s)'
              }}
            >
              <small>{note.savedAt.toLocaleString()}</small>
              <pre style={{ margin: 0, whiteSpace: 'pre-wrap' }}>{note.content}</pre>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Diff Viewer

```jsx
function DiffTextarea({ original, modified, onModifiedChange }) {
  const [showDiff, setShowDiff] = useState(false);

  return (
    <div>
      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 'var(--spacing-m)' }}>
        <div>
          <h4>Original</h4>
          <TextareaInput
            value={original}
            readOnly
            rows={10}
            style={{ backgroundColor: 'var(--light-color)' }}
          />
        </div>
        
        <div>
          <h4>Modified</h4>
          <TextareaInput
            value={modified}
            onChange={(e) => onModifiedChange(e.target.value)}
            rows={10}
          />
        </div>
      </div>
      
      <Button 
        onClick={() => setShowDiff(!showDiff)}
        style={{ marginTop: 'var(--spacing-m)' }}
      >
        {showDiff ? 'Hide' : 'Show'} Differences
      </Button>
      
      {showDiff && (
        <div style={{ marginTop: 'var(--spacing-m)' }}>
          {/* Diff visualization */}
        </div>
      )}
    </div>
  );
}
```

### Template Variables

```jsx
function TemplateTextarea({ variables }) {
  const [template, setTemplate] = useState('');
  const textareaRef = useRef(null);

  const insertVariable = (variable) => {
    const textarea = textareaRef.current;
    if (!textarea) return;

    const start = textarea.selectionStart;
    const end = textarea.selectionEnd;
    const newValue = 
      template.substring(0, start) + 
      `{{${variable}}}` + 
      template.substring(end);
    
    setTemplate(newValue);
    
    // Set cursor after inserted variable
    setTimeout(() => {
      const newPosition = start + variable.length + 4; // +4 for {{ }}
      textarea.setSelectionRange(newPosition, newPosition);
      textarea.focus();
    }, 0);
  };

  return (
    <div>
      <div style={{ marginBottom: 'var(--spacing-s)' }}>
        <span style={{ marginRight: 'var(--spacing-s)' }}>Insert variable:</span>
        {variables.map(v => (
          <Button
            key={v}
            size="s"
            onClick={() => insertVariable(v)}
            style={{ marginRight: 'var(--spacing-xs)' }}
          >
            {v}
          </Button>
        ))}
      </div>
      
      <TextareaInput
        ref={textareaRef}
        value={template}
        onChange={(e) => setTemplate(e.target.value)}
        placeholder="Use {{variable}} syntax..."
        rows={6}
      />
      
      <div style={{ marginTop: 'var(--spacing-s)' }}>
        <h4>Preview:</h4>
        <div style={{
          padding: 'var(--spacing-m)',
          backgroundColor: 'var(--light-color)',
          borderRadius: 'var(--border-radius-s)'
        }}>
          {template.replace(/\{\{(\w+)\}\}/g, '<mark>$1</mark>')}
        </div>
      </div>
    </div>
  );
}
```

## Form Integration

### With FieldWrapper

```jsx
import { FieldWrapper, FormLabel, TextareaInput, FieldHint } from 'datocms-react-ui';

function CustomTextareaField({ label, hint, error, ...textareaProps }) {
  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <TextareaInput error={!!error} {...textareaProps} />
      {hint && <FieldHint>{hint}</FieldHint>}
      {error && <FieldError>{error}</FieldError>}
    </FieldWrapper>
  );
}
```

### In a Form

```jsx
function CommentForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    comment: ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <Form onSubmit={handleSubmit}>
      <FormLabel htmlFor="name">Name</FormLabel>
      <TextInput
        id="name"
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        required
      />
      
      <FormLabel htmlFor="email">Email</FormLabel>
      <TextInput
        id="email"
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        required
      />
      
      <FormLabel htmlFor="comment">Comment</FormLabel>
      <TextareaInput
        id="comment"
        value={formData.comment}
        onChange={(e) => setFormData({ ...formData, comment: e.target.value })}
        rows={5}
        required
      />
      
      <Button type="submit" buttonType="primary">
        Submit Comment
      </Button>
    </Form>
  );
}
```

## Accessibility

- Native textarea element with full keyboard support
- Works with screen readers
- Supports standard HTML attributes
- Error state communicated visually
- Can be labeled with external labels

## TypeScript

```typescript
import { TextareaInput } from 'datocms-react-ui';
import { ChangeEvent } from 'react';

interface CustomTextareaProps {
  value: string;
  onChange: (value: string) => void;
  error?: boolean;
  maxLength?: number;
  rows?: number;
}

function CustomTextarea({ 
  value, 
  onChange,
  error,
  maxLength,
  rows = 4
}: CustomTextareaProps) {
  const handleChange = (e: ChangeEvent<HTMLTextAreaElement>) => {
    const newValue = e.target.value;
    if (!maxLength || newValue.length <= maxLength) {
      onChange(newValue);
    }
  };

  return (
    <TextareaInput
      value={value}
      onChange={handleChange}
      error={error}
      rows={rows}
      maxLength={maxLength}
    />
  );
}
```

## Best Practices

1. **Controlled component**: Always use controlled mode with value/onChange
2. **Appropriate rows**: Set rows based on expected content
3. **Error handling**: Use the error prop for validation states
4. **Accessibility**: Ensure proper labeling with htmlFor
5. **Placeholder text**: Provide helpful examples
6. **Character limits**: Consider showing character count

## Related Components

- [TextareaField](./TextareaField.md) - Complete textarea with label
- [TextInput](./TextInput.md) - Single-line text input
- [FieldWrapper](./FieldWrapper.md) - For custom field layouts
- [FormLabel](./FormLabel.md) - For labeling