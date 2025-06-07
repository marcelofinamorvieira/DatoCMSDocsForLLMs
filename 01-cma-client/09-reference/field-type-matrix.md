# Field Type Matrix

## Field Types Overview

| Field Type | API Key | Value Type | Localized | Default | Appearance Options |
|------------|---------|------------|-----------|---------|-------------------|
| Single-line string | `string` | `string` | ✅ | ✅ | Multiple |
| Multi-line text | `text` | `string` | ✅ | ✅ | Textarea, Markdown |
| Boolean | `boolean` | `boolean` | ✅ | ✅ | Switch, Radio |
| Integer | `integer` | `number` | ✅ | ✅ | Input, Slider |
| Float | `float` | `number` | ✅ | ✅ | Input, Slider |
| Date | `date` | `string` (ISO) | ✅ | ✅ | Calendar |
| DateTime | `datetime` | `string` (ISO) | ✅ | ✅ | Calendar + Time |
| Color | `color` | `object` | ✅ | ✅ | Color picker |
| JSON | `json` | `object` | ✅ | ✅ | JSON editor |
| Location | `lat_lon` | `object` | ✅ | ❌ | Map picker |
| SEO | `seo` | `object` | ✅ | ❌ | SEO form |
| Slug | `slug` | `string` | ✅ | ❌ | Slug input |
| Single asset | `file` | `object` | ✅ | ❌ | Media picker |
| Asset gallery | `gallery` | `array` | ✅ | ❌ | Gallery picker |
| Single link | `link` | `string` | ✅ | ❌ | Item selector |
| Multiple links | `links` | `array` | ✅ | ❌ | Multi selector |
| Modular content | `rich_text` | `array` | ✅ | ❌ | Block editor |
| Single block | `single_block` | `object` | ✅ | ❌ | Block selector |
| Structured text | `structured_text` | `object` | ✅ | ❌ | Rich editor |

## Value Type Examples

### String Fields
```javascript
// string
title: "Hello World"

// text
description: "Long text content..."

// slug
url_slug: "hello-world"
```

### Number Fields
```javascript
// integer
quantity: 42

// float
price: 19.99
```

### Boolean Field
```javascript
// boolean
published: true
featured: false
```

### Date/Time Fields
```javascript
// date
publish_date: "2024-01-15"

// datetime
event_start: "2024-01-15T14:30:00Z"
```

### Object Fields
```javascript
// color
brand_color: {
  red: 255,
  green: 87,
  blue: 51,
  alpha: 1,
  hex: "#FF5733"
}

// lat_lon
location: {
  latitude: 40.7128,
  longitude: -74.0060
}

// json
metadata: {
  custom: "data",
  nested: { values: [1, 2, 3] }
}

// seo
seo_settings: {
  title: "Page Title",
  description: "Meta description",
  image: { upload_id: "123" }
}
```

### Asset Fields
```javascript
// file
featured_image: {
  upload_id: "upload-123",
  alt: "Alt text",
  title: "Image title"
}

// gallery
images: [
  { upload_id: "upload-1", alt: "Image 1" },
  { upload_id: "upload-2", alt: "Image 2" }
]
```

### Relationship Fields
```javascript
// link
author: "author-123"

// links
categories: ["cat-1", "cat-2", "cat-3"]

// rich_text (modular content)
sections: [
  {
    item_type: "hero_section",
    headline: "Welcome"
  },
  {
    item_type: "text_section",
    content: "Content..."
  }
]

// single_block
featured_component: {
  item_type: "cta_block",
  title: "Get Started",
  button_text: "Sign Up Now"
}
```

## Validator Compatibility

| Field Type | Required | Unique | Length | Format | Range | Size | Custom |
|------------|----------|--------|--------|--------|-------|------|--------|
| string | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| text | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| boolean | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| integer | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ |
| float | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ |
| date | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| datetime | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| color | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| json | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| lat_lon | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| seo | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| slug | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| file | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| gallery | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| link | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| links | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| rich_text | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| single_block | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| structured_text | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

## Appearance Options

### Text Fields
- **string**: Text input, URL input, Select, Radio
- **text**: Textarea, Markdown editor, WYSIWYG

### Number Fields
- **integer/float**: Number input, Slider, Rating

### Boolean
- **boolean**: Switch, Radio buttons

### Date/Time
- **date/datetime**: Calendar picker, Custom format

### Media
- **file**: Default picker, Custom uploader
- **gallery**: Grid view, List view

### Relationships
- **link/links**: Dropdown, Modal picker, Inline
- **rich_text**: Nested blocks, Tabs
- **single_block**: Framed selector, Frameless selector

## Field Selection Guide

| Need | Use Field Type | Example |
|------|----------------|---------|
| Short text (< 255 chars) | `string` | Title, Name |
| Long text | `text` | Description, Bio |
| Rich formatted text | `structured_text` | Article content |
| Yes/No choice | `boolean` | Published, Featured |
| Whole numbers | `integer` | Quantity, Views |
| Decimals | `float` | Price, Rating |
| Dates | `date` | Birth date |
| Date + time | `datetime` | Event time |
| Colors | `color` | Brand color |
| Coordinates | `lat_lon` | Store location |
| URL-friendly text | `slug` | URL path |
| Single image/file | `file` | Avatar, PDF |
| Multiple images | `gallery` | Product photos |
| Reference one item | `link` | Author |
| Reference many items | `links` | Categories |
| Array of blocks | `rich_text` | Page sections |
| Single block | `single_block` | Hero section |
| Arbitrary data | `json` | Settings |
| SEO metadata | `seo` | Page SEO |

## Special Considerations

### Localized Fields
- All field types support localization
- Each locale stores separate value
- Can have different validation per locale

### Required Fields
- `required` validator works on all types
- For relationships, means must select at least one
- For gallery/links, can combine with `size` validator

### Unique Fields
- Only on: `string`, `integer`, `float`, `slug`
- Enforced across all items of same model
- Case-sensitive for strings

### Default Values
- Not available for: media, relationships, location, SEO
- Must match field type
- Applied only on creation