# TextareaField Component

A multi-line text input field that combines a label, textarea, hint text, and error handling in one component, matching DatoCMS's design system.

## Import

```jsx
import { TextareaField } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | - | Field label |
| `value` | `string` | - | Current field value |
| `onChange` | `(value: string) => void` | - | Change handler |
| `placeholder` | `string` | - | Placeholder text |
| `hint` | `string` | - | Help text below field |
| `error` | `string` | - | Error message |
| `required` | `boolean` | `false` | Mark as required |
| `disabled` | `boolean` | `false` | Disable the field |
| `readOnly` | `boolean` | `false` | Make field read-only |
| `rows` | `number` | `4` | Number of visible rows |
| `maxLength` | `number` | - | Maximum character length |
| `autoResize` | `boolean` | `false` | Auto-resize to content |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { TextareaField, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [description, setDescription] = useState('');

  return (
    <Canvas ctx={ctx}>
      <TextareaField
        label="Description"
        value={description}
        onChange={setDescription}
        placeholder="Enter a detailed description..."
        rows={6}
      />
    </Canvas>
  );
}
```

## Common Patterns

### With Character Counter

```jsx
function TextareaWithCounter() {
  const [value, setValue] = useState('');
  const maxLength = 500;

  return (
    <TextareaField
      label="Bio"
      value={value}
      onChange={setValue}
      maxLength={maxLength}
      hint={`${value.length}/${maxLength} characters`}
      error={value.length > maxLength ? 'Text is too long' : undefined}
      rows={4}
    />
  );
}
```

### Auto-Resizing Textarea

```jsx
function AutoResizingTextarea() {
  const [content, setContent] = useState('');

  return (
    <TextareaField
      label="Content"
      value={content}
      onChange={setContent}
      autoResize={true}
      rows={3} // Minimum rows
      placeholder="Start typing... the field will expand as needed"
    />
  );
}
```

### Form with Multiple Textareas

```jsx
function ContentForm({ ctx }) {
  const [formData, setFormData] = useState({
    summary: '',
    content: '',
    notes: ''
  });

  const updateField = (field, value) => {
    setFormData({ ...formData, [field]: value });
  };

  return (
    <Form onSubmit={handleSubmit}>
      <TextareaField
        label="Summary"
        value={formData.summary}
        onChange={(value) => updateField('summary', value)}
        hint="Brief overview (2-3 sentences)"
        rows={2}
        maxLength={200}
        required
      />
      
      <TextareaField
        label="Main Content"
        value={formData.content}
        onChange={(value) => updateField('content', value)}
        placeholder="Enter the main content here..."
        rows={10}
        required
      />
      
      <TextareaField
        label="Internal Notes"
        value={formData.notes}
        onChange={(value) => updateField('notes', value)}
        hint="These notes are not public"
        rows={3}
      />
      
      <Button type="submit" buttonType="primary">
        Save Content
      </Button>
    </Form>
  );
}
```

## Advanced Examples

### Markdown Editor

```jsx
function MarkdownTextarea({ label, value, onChange }) {
  const [preview, setPreview] = useState(false);

  return (
    <div>
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: 'var(--spacing-s)'
      }}>
        <FormLabel>{label}</FormLabel>
        <Button 
          size="s" 
          onClick={() => setPreview(!preview)}
        >
          {preview ? 'Edit' : 'Preview'}
        </Button>
      </div>
      
      {preview ? (
        <div style={{
          minHeight: '200px',
          padding: 'var(--spacing-m)',
          backgroundColor: 'var(--light-color)',
          borderRadius: 'var(--border-radius-s)',
          border: '1px solid var(--border-color)'
        }}>
          <ReactMarkdown>{value || '*No content*'}</ReactMarkdown>
        </div>
      ) : (
        <TextareaField
          value={value}
          onChange={onChange}
          placeholder="Enter markdown text..."
          hint="Supports **bold**, *italic*, [links](url), and more"
          rows={10}
        />
      )}
    </div>
  );
}
```

### JSON Editor

```jsx
function JSONTextarea({ label, value, onChange }) {
  const [error, setError] = useState('');
  const [formatted, setFormatted] = useState(value);

  const handleChange = (newValue) => {
    setFormatted(newValue);
    setError('');
    
    try {
      JSON.parse(newValue);
      onChange(newValue);
    } catch (e) {
      setError('Invalid JSON: ' + e.message);
    }
  };

  const formatJSON = () => {
    try {
      const parsed = JSON.parse(formatted);
      const pretty = JSON.stringify(parsed, null, 2);
      setFormatted(pretty);
      onChange(pretty);
      setError('');
    } catch (e) {
      setError('Cannot format invalid JSON');
    }
  };

  return (
    <div>
      <TextareaField
        label={label}
        value={formatted}
        onChange={handleChange}
        error={error}
        rows={10}
        style={{ fontFamily: 'monospace' }}
      />
      <Button 
        onClick={formatJSON}
        size="s"
        style={{ marginTop: 'var(--spacing-xs)' }}
      >
        Format JSON
      </Button>
    </div>
  );
}
```

### Template Editor

```jsx
function TemplateEditor({ templates, onSave }) {
  const [selectedTemplate, setSelectedTemplate] = useState('');
  const [content, setContent] = useState('');
  const [variables, setVariables] = useState([]);

  const loadTemplate = (templateId) => {
    const template = templates.find(t => t.id === templateId);
    if (template) {
      setContent(template.content);
      setVariables(template.variables || []);
    }
  };

  const insertVariable = (variable) => {
    setContent(content + `{{${variable}}}`);
  };

  return (
    <div>
      <SelectField
        label="Template"
        value={selectedTemplate}
        onChange={(value) => {
          setSelectedTemplate(value);
          loadTemplate(value);
        }}
      >
        <option value="">Select a template</option>
        {templates.map(t => (
          <option key={t.id} value={t.id}>{t.name}</option>
        ))}
      </SelectField>
      
      {selectedTemplate && (
        <>
          <div style={{ marginBottom: 'var(--spacing-s)' }}>
            <FormLabel>Available Variables</FormLabel>
            <div style={{ display: 'flex', gap: 'var(--spacing-xs)', flexWrap: 'wrap' }}>
              {variables.map(v => (
                <Button
                  key={v}
                  size="s"
                  onClick={() => insertVariable(v)}
                >
                  {`{{${v}}}`}
                </Button>
              ))}
            </div>
          </div>
          
          <TextareaField
            label="Template Content"
            value={content}
            onChange={setContent}
            rows={10}
            hint="Use {{variable}} syntax for dynamic content"
          />
          
          <Button 
            onClick={() => onSave(selectedTemplate, content)}
            buttonType="primary"
          >
            Save Template
          </Button>
        </>
      )}
    </div>
  );
}
```

### Code Editor with Syntax Highlighting

```jsx
function CodeTextarea({ language = 'javascript', value, onChange }) {
  const [focused, setFocused] = useState(false);

  return (
    <div style={{ position: 'relative' }}>
      <TextareaField
        label={`Code (${language})`}
        value={value}
        onChange={onChange}
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        rows={15}
        style={{
          fontFamily: 'monospace',
          fontSize: '14px',
          tabSize: 2
        }}
        placeholder={`Enter ${language} code...`}
      />
      
      {!focused && value && (
        <div style={{
          position: 'absolute',
          top: '40px', // Account for label
          left: '1px',
          right: '1px',
          bottom: '1px',
          padding: 'var(--spacing-s)',
          pointerEvents: 'none',
          overflow: 'hidden'
        }}>
          <SyntaxHighlighter language={language}>
            {value}
          </SyntaxHighlighter>
        </div>
      )}
    </div>
  );
}
```

### Collaborative Notes

```jsx
function CollaborativeNotes({ ctx, itemId }) {
  const [notes, setNotes] = useState('');
  const [lastSaved, setLastSaved] = useState(null);
  const [saving, setSaving] = useState(false);

  // Auto-save functionality
  useEffect(() => {
    const timer = setTimeout(() => {
      if (notes && notes !== lastSaved) {
        saveNotes();
      }
    }, 2000); // Save after 2 seconds of inactivity

    return () => clearTimeout(timer);
  }, [notes]);

  const saveNotes = async () => {
    setSaving(true);
    try {
      await ctx.updateItem(itemId, { notes });
      setLastSaved(notes);
    } finally {
      setSaving(false);
    }
  };

  return (
    <div>
      <TextareaField
        label="Collaborative Notes"
        value={notes}
        onChange={setNotes}
        placeholder="Add notes about this item..."
        hint={
          saving ? 'Saving...' : 
          lastSaved === notes ? 'All changes saved' : 
          'Unsaved changes'
        }
        rows={6}
      />
      
      <div style={{ 
        marginTop: 'var(--spacing-s)',
        fontSize: 'var(--font-size-s)',
        color: 'var(--muted-color)'
      }}>
        Last edited by {ctx.currentUser.name}
      </div>
    </div>
  );
}
```

### Rich Text Alternative

```jsx
function RichTextarea({ value, onChange }) {
  const [formatting, setFormatting] = useState({
    bold: false,
    italic: false,
    underline: false
  });

  const applyFormatting = (type) => {
    const textarea = document.getElementById('rich-textarea');
    const start = textarea.selectionStart;
    const end = textarea.selectionEnd;
    const selectedText = value.substring(start, end);
    
    if (selectedText) {
      let formattedText = selectedText;
      switch(type) {
        case 'bold':
          formattedText = `**${selectedText}**`;
          break;
        case 'italic':
          formattedText = `*${selectedText}*`;
          break;
        case 'link':
          formattedText = `[${selectedText}](url)`;
          break;
      }
      
      const newValue = value.substring(0, start) + formattedText + value.substring(end);
      onChange(newValue);
    }
  };

  return (
    <div>
      <div style={{ 
        display: 'flex', 
        gap: 'var(--spacing-xs)',
        marginBottom: 'var(--spacing-s)'
      }}>
        <Button size="s" onClick={() => applyFormatting('bold')}>
          <strong>B</strong>
        </Button>
        <Button size="s" onClick={() => applyFormatting('italic')}>
          <em>I</em>
        </Button>
        <Button size="s" onClick={() => applyFormatting('link')}>
          Link
        </Button>
      </div>
      
      <TextareaField
        id="rich-textarea"
        label="Content"
        value={value}
        onChange={onChange}
        rows={10}
        hint="Select text and use formatting buttons"
      />
    </div>
  );
}
```

## Validation Examples

```jsx
function ValidatedTextarea() {
  const [value, setValue] = useState('');
  const [errors, setErrors] = useState([]);

  const validate = (text) => {
    const newErrors = [];
    
    if (!text.trim()) {
      newErrors.push('Content is required');
    }
    if (text.length < 50) {
      newErrors.push('Minimum 50 characters required');
    }
    if (text.length > 1000) {
      newErrors.push('Maximum 1000 characters allowed');
    }
    if (!/[.!?]$/.test(text.trim())) {
      newErrors.push('Must end with punctuation');
    }
    
    setErrors(newErrors);
  };

  return (
    <TextareaField
      label="Article Summary"
      value={value}
      onChange={(val) => {
        setValue(val);
        validate(val);
      }}
      error={errors[0]} // Show first error
      hint={errors.length > 1 ? `${errors.length} issues found` : undefined}
      rows={5}
      required
    />
  );
}
```

## Styling

### Custom Styling

```jsx
<TextareaField
  label="Custom Styled"
  value={value}
  onChange={setValue}
  style={{
    backgroundColor: 'var(--light-color)',
    fontFamily: 'Georgia, serif',
    fontSize: '16px'
  }}
  className="custom-textarea"
  rows={8}
/>
```

## Accessibility

- Proper label association
- ARIA attributes for errors and hints
- Keyboard navigation support
- Screen reader announcements
- Required field indication

## TypeScript

```typescript
import { TextareaField } from 'datocms-react-ui';

interface FormData {
  description: string;
  notes: string;
}

interface ContentFormProps {
  data: FormData;
  onChange: (data: FormData) => void;
  errors?: Partial<Record<keyof FormData, string>>;
}

function ContentForm({ data, onChange, errors }: ContentFormProps) {
  const updateField = (field: keyof FormData, value: string) => {
    onChange({ ...data, [field]: value });
  };

  return (
    <>
      <TextareaField
        label="Description"
        value={data.description}
        onChange={(value) => updateField('description', value)}
        error={errors?.description}
        rows={4}
        required
      />
      
      <TextareaField
        label="Notes"
        value={data.notes}
        onChange={(value) => updateField('notes', value)}
        error={errors?.notes}
        rows={3}
      />
    </>
  );
}
```

## Best Practices

1. **Appropriate rows**: Set rows based on expected content length
2. **Character limits**: Show limits and current count
3. **Placeholder text**: Provide helpful examples
4. **Auto-save**: Consider auto-saving for longer content
5. **Rich formatting**: Offer formatting options when needed
6. **Validation feedback**: Show errors clearly

## Related Components

- [TextareaInput](./TextareaInput.md) - Just the textarea input
- [TextField](./TextField.md) - Single-line text input
- [Form](./Form.md) - Form container
- [FieldError](./FieldError.md) - Error messages