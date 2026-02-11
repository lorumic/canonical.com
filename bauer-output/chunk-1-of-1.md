# BAU Copy Update Implementation Instructions

You are assisting with implementing copy suggestions from a Google Doc into a web project that uses the Vanilla Framework from Canonical. The feedback is provided as structured suggestions in JSON format. You are only required to update text fields. 

Your task is to accurately apply these suggestions to the correct files in the project repository. Once you read and understand this document, implement all of the suggestions in the provided JSON data, and follow the instructions carefully.

## Project Context

- **Framework**: Vanilla Framework (https://vanillaframework.io/)
- **Template Engine**: Jinja2
- **Repository**: Current working directory (ensure you're in the target repo)
- **Branch**: Currently active branch
- **Document**: Copy of /solutions/AI/infrastructure

## Finding Target Files

The target file path is specified in the metadata as: **https://canonical.com/solutions/ai/infrastructure**

### Path Resolution Rules

1. **For most pages**: Create/edit a file with the appropriate page name
   - Example: `ubuntu.com/desktop/upcoming-features` → `templates/desktop/upcoming-features.html`

2. **For index pages**: If the URL path matches a folder name
   - Example: `ubuntu.com/desktop` → `templates/desktop/index.html`
   - Create the folder if it doesn't exist

3. **Nested paths**: Create all necessary parent directories
   - Example: `ubuntu.com/engage/resources/guide` → `templates/engage/resources/guide.html`

### File Location Algorithm

```
Given URL: domain.com/path/to/page

1. Remove domain: /path/to/page
2. Check if file exists at: templates/path/to/page.html
3. If not, check: templates/path/to/page/index.html
4. If creating new file:
   - Create directories: templates/path/to/
   - Create file: page.html (or index.html if path ends with existing folder name)
```

## Understanding the Suggestions JSON Schema

Each suggestion in the JSON array is a **LocationGroupedSuggestions** object with this structure:

```json
{
  "location": {
    "section": "Body",              // Section of document (Body, Header, Footer)
    "parent_heading": "Section Name", // Optional: Nearest heading above
    "heading_level": 2,               // Optional: Heading level (1-6)
    "in_table": false,                // Whether suggestion is in a table
    "in_metadata": false,             // True if suggestion comes from the metadata table
    "table": {                        // Optional: Table context if in_table is true
      "table_title": "Pattern Name",  // Pattern name (Hero, Equal Heights, etc.)
      "row_index": 1,
      "column_index": 2,
      "column_header": "Header",
      "row_header": "Row Label"
    }
  },
  "suggestions": [                    // Array of suggestions for this location
    {
      "id": "suggestion-id",
      "anchor": {
        "preceding_text": "exact text before",
        "following_text": "exact text after"
      },
      "change": {
        "type": "insert|delete|replace",
        "original_text": "text to remove/replace",  // Empty for inserts
        "new_text": "text to add/replace with"      // Empty for deletes
      },
      "verification": {
        "text_before_change": "combined before state",
        "text_after_change": "combined after state"
      },
      "position": {
        "start_index": 123,     // Character index in the document before change. Do not use this to locate text, it's for reference only.
        "end_index": 456        // Character index in the document before change. Do not use this to locate text, it's for reference only. 
      },
      "atomic_count": 1                 // Number of atomic operations merged
    }
  ]
}
```

## Applying Changes

Process the suggestions **one location at a time, in order**. For each location:

1. **Read the location context**: Understand where in the document this is
2. **Process each suggestion** in the `suggestions` array sequentially
3. **Apply each change** following the process below
4. **Verify** before moving to the next suggestion

### Metadata Table Suggestions

If `location.in_metadata` is true, the change came from the document metadata table and likely
does not appear verbatim in the target HTML content. Instead, map the change to metadata tags
in the repo that mirror the metadata table entries.

Use this process:
1. **Identify the metadata key**:
   - Prefer `location.table.row_header` (the left column label in the metadata table).
   - Use `location.table.column_header` only if `row_header` is empty.
2. **Find the tag** in the repo:
   - Search for tags or markers that match the metadata key (they mirror each table entry).
   - Use this strict match order:
     1. Exact text match (`Page title` == `Page title`)
     2. Case-insensitive match (`Page title` == `page title`)
     3. Normalized text match (trim + collapse spaces + remove punctuation)
     4. Snake case match (`Page title` -> `page_title`)
   - Stop at the first successful match; do not keep searching looser matches afterward.
   - Look for HTML comments, data attributes, front-matter fields, or template variables.
3. **Apply the change**:
   - Update the tag value to match the suggestion's `new_text`.
   - Do not use anchor text matching for metadata suggestions.
4. **Verify**:
   - Confirm the tag value matches `text_after_change`.

If multiple files contain the same metadata tag, prefer the file matching `https://canonical.com/solutions/ai/infrastructure`.
If the tag is missing, report it as an error and continue.

### Application Process

For each suggestion:

1. **Locate the text**:
   - Search for: `{preceding_text}{original_text}{following_text}`
   - The anchor texts are exact strings from the document

2. **Apply the change** based on type:
   - **insert**: Add `new_text` between `preceding_text` and `following_text`
   - **delete**: Remove `original_text`, keeping anchors intact
   - **replace**: Substitute `original_text` with `new_text`

3. **Verify**:
   - Confirm the resulting text matches `text_after_change`
   - If mismatch, report the discrepancy

### Important Notes

- **Preserve formatting**: Maintain HTML structure, indentation, and styling
- **Exact matching**: Anchor texts are precise - use them to find locations
- **Order matters**: Process suggestions in the order provided
- **Pattern awareness**: If `table_title` indicates a Vanilla pattern, consult the patterns reference below
- **Metadata tags**: For `in_metadata` suggestions, update the matching tag in the target repo instead of searching for anchors
- **Style changes**: Some suggestions may be style-only changes (e.g., making text bold, adding emphasis). Use appropriate Vanilla Framework classes and HTML to apply these changes.
- **Section deletions**: It is expected that some suggestions involve removing entire sections, this is acceptable behavior, ensure proper HTML structure and semantics are maintained. 

## Vanilla Framework Patterns

This section is added so you can understand the vanilla patterns you may encounter. Do NOT change any pattern, and report to the user any pattern mismatches.

To identify a pattern from the suggestion use `table_title` in location metadata and match with the corresponding pattern in the Vanilla Patterns Reference section that follows these instructions.

**Note**: The complete Vanilla Framework Patterns Reference appears immediately after these instructions and before the suggestions data.

## Error Handling

If you encounter issues:

1. **File not found**:
   - Check if the path needs index.html instead
   - Verify parent directories exist
   - Report if the URL structure is ambiguous

2. **Anchor text not found**:
   - The file may have been modified since the doc was created
   - Report the missing anchor and suggestion details
   - Ask for manual verification

3. **Verification mismatch**:
   - Report expected vs actual text
   - Indicate which suggestion failed
   - Continue with remaining suggestions

## Document Structure

This prompt is organized in the following order:

1. **These instructions** (what you're reading now)
2. **Vanilla Framework Patterns Reference** (reference material for implementing patterns)
3. **Suggestions Data** (JSON array of changes to implement)

## Processing Instructions

**Chunk 1 of 1**

Process the suggestions data at the end of this document one location at a time. After processing ALL locations in this chunk, report:
- Number of locations processed
- Number of successful changes
- Any errors or issues encountered


---

# Vanilla patterns

This file summarizes common Vanilla patterns and how to use them from Jinja macros. Each pattern below contains:
- purpose (one line),
- required params / slots,
- minimal Jinja import + usage examples,
- short configuration notes.

You should import all required macros at the beginning of the Jinja template before using them.

Table of contents
- [Hero pattern](#hero-pattern)
- [Equal heights](#equal-heights)
- [Text Spotlight](#text-spotlight)
- [Logo section](#logo-section)
- [Tab section](#tab-section)
- [Tiered list](#tiered-list)
- [Basic section](#basic-section)

---

## Hero pattern

Purpose: prominent banner (h1) with optional subtitle, description, CTA and image(s).

Key points
- Required param: `title_text` (renders as `h1`).
- Layouts: `'50/50'`, `'50/50-full-width-image'`, `'75/25'`, `'25/75'`, `'fallback'`.
- Flags: `is_split_on_medium` (bool), `display_blank_signpost_image_space` (bool).
- Slots (callable): `description`, `cta`, `image`, `signpost_image`.

Jinja import
```/dev/null/hero-import.jinja#L1-3
{% from "_macros/vf_hero.jinja" import vf_hero %}
```

Minimal usage
```/dev/null/hero-example.jinja#L1-20
{% from "_macros/vf_hero.jinja" import vf_hero %}

{% call(slot) vf_hero(
  title_text='Welcome to our product',
  subtitle_text='Short subtitle',
  layout='50/50',
  is_split_on_medium=true
) %}
  {% call(description) %}<p>Short description.</p>{% endcall %}
  {% call(cta) %}<a class="p-button" href="/signup">Get started</a>{% endcall %}
  {% call(image) %}<img src="/assets/hero.jpg" alt="Hero" />{% endcall %}
{% endcall %}
```

Notes
- For `25/75` provide `signpost_image` or set `display_blank_signpost_image_space=true`.
- For full-width images use `50/50-full-width-image` or place an image container at the same level as the grid columns.
- Import full Vanilla SCSS for consistent styling.

---

## Equal heights

Purpose: grid of item tiles with consistent heights (useful for features, cards).

Key points
- Required params: `title_text`, `items` (Array<Object>).
- Common item fields: `title_text`, `title_link_attrs`, `description_html`, `image_html`, `cta_html`.
- Image aspect controls: `image_aspect_ratio_small`, `image_aspect_ratio_medium`, `image_aspect_ratio_large`.
- Option: `highlight_images` (boolean) to style illustrations.

Jinja import
```/dev/null/equal-heights-import.jinja#L1-3
{% from "_macros/vf_equal-heights.jinja" import vf_equal_heights %}
```

Minimal usage (using call syntax with slots)
```/dev/null/equal-heights-example.jinja#L1-40
{% from "_macros/vf_equal-heights.jinja" import vf_equal_heights %}

{% call(slot) vf_equal_heights(
  title_text="Keep this heading to 2 lines on large screens.",
  attrs={ "id": "4-columns-responsive" },
  subtitle_text="Ensure the right hand side of this 50/50 split is taller than the left hand side (heading) on its left. This includes the subtitle and description.",
  items=[
    {
      "title_text": "A strong hardware ecosystem",
      "image_html":  "<img src='https://assets.ubuntu.com/v1/ff6a068d-kernelt-vanilla-ehp-1.png' class='p-image-container__image' width='284' height='426' alt='Kernelt' />",
      "description_html": "<p>We enable Ubuntu Core with the best ODMs and silicon vendors in the world. We continuously test it on leading IoT and edge devices and hardware.</p>",
      "cta_html": "<a href='#'>Browse all certified hardware&nbsp;&rsaquo;</a>"
    },
    {
      "title_text": "A strong hardware ecosystem",
      "image_html":  "<img src='https://assets.ubuntu.com/v1/7aa4ed28-kernelt-vanilla-ehp-2.png' class='p-image-container__image' width='284' height='426' alt='Kernelt' />",
      "description_html": "<p>We enable Ubuntu Core with the best ODMs and silicon vendors in the world. We continuously test it on leading IoT and edge devices and hardware.</p>",
      "cta_html": "<a href='#'>Browse all certified hardware&nbsp;&rsaquo;</a>"
    },
    {
      "title_text": "A strong hardware ecosystem",
      "image_html":  "<img src='https://assets.ubuntu.com/v1/4936d43a-kernelt-vanilla-ehp-3.png' class='p-image-container__image' width='284' height='426' alt='Kernelt' />",
      "description_html": "<p>We enable Ubuntu Core with the best ODMs and silicon vendors in the world. We continuously test it on leading IoT and edge devices and hardware.</p>",
      "cta_html": "<a href='#'>Browse all certified hardware&nbsp;&rsaquo;</a>"
    },
    {
      "title_text": "A strong hardware ecosystem",
      "image_html":  "<img src='https://assets.ubuntu.com/v1/bbe7b062-kernelt-vanilla-ehp-4.png' class='p-image-container__image' width='284' height='426' alt='Kernelt' />",
      "description_html": "<p>We enable Ubuntu Core with the best ODMs and silicon vendors in the world. We continuously test it on leading IoT and edge devices and hardware.</p>",
      "cta_html": "<a href='#'>Browse all certified hardware&nbsp;&rsaquo;</a>"
    }
  ]
) %}
{% endcall %}
```

Notes
- Prefer consistent properties across `items` for visual rhythm.
- If number of items is divisible by 4/3, layout adjusts to 4/3 columns on large screens.
- For the parent `title_text` use the text from the first suggestion in the given location, if it is much shorter than the others.

---

## Text Spotlight

Purpose: callout list of short items (2–7), used to highlight benefits or actions.

Key points
- Required params: `title_text`, `list_items` (Array<string>).
- Option: `item_heading_level` (2 or 4) — controls item styling.

Jinja import
```/dev/null/text-spotlight-import.jinja#L1-3
{% from "_macros/vf_text-spotlight.jinja" import vf_text_spotlight %}
```

Minimal usage
```/dev/null/text-spotlight-example.jinja#L1-12
{% from "_macros/vf_text-spotlight.jinja" import vf_text_spotlight %}

{{ vf_text_spotlight(
  title_text='Why choose us',
  list_items=['Fast setup','Secure by default','Enterprise support'],
  item_heading_level=2
) }}
```

Notes
- `list_items` may contain HTML strings if needed.
- Use `item_heading_level=4` for more compact item headings.

---

## Logo section (aka logo cloud)

Purpose: heading + optional description + a block of logos or CTA blocks (use for partner/client logos).

Key points
- Required param: `title` (Object with `text` and optional `link_attrs`).
- Required param: `blocks` — Array of block objects (`type: "logo-block"` or `type: "cta-block"`).
- Slot: `description` (optional) for descriptive paragraphs.
- `padding` can be `'default'` or `'deep'`.

Jinja import
```/dev/null/logo-section-import.jinja#L1-3
{% from "_macros/vf_logo-section.jinja" import vf_logo_section %}
```

Minimal usage (using call syntax with slots)
```/dev/null/logo-section-example.jinja#L1-60
{% from "_macros/vf_logo-section.jinja" import vf_logo_section %}

{% call(slot) vf_logo_section(
  title={
    "text": "The quick brown fox jumps over the lazy dog"
  },
  blocks=[
    {
      "type": "cta-block",
      "item": {
        "primary": {
          "content_html": "Primary Button",
          "attrs": {
            "href": "#"
          }
        },
        "link": {
          "content_html": "Lorem ipsum dolor sit amet ›",
          "attrs": {
            "href": "#"
          }
        }
      }
    },
    {
      "type": "logo-block",
      "item": {
        "logos": [
          {
            "src": "https://assets.ubuntu.com/v1/38fdfd23-Dell-logo.png",
            "alt": "Dell Technologies"
          },
          {
            "src": "https://assets.ubuntu.com/v1/cd5f636a-hp-logo.png",
            "alt": "Hewlett Packard"
          },
          {
            "src": "https://assets.ubuntu.com/v1/f90702cd-lenovo-logo.png",
            "alt": "Lenovo"
          },
          {
            "src": "https://assets.ubuntu.com/v1/2ef3c028-amazon-web-services-logo.png",
            "alt": "Amazon Web Services"
          },
          {
            "src": "https://assets.ubuntu.com/v1/cb7ef8ac-ibm-cloud-logo.png",
            "alt": "IBM Cloud"
          },
          {
            "src": "https://assets.ubuntu.com/v1/210f44e4-microsoft-azure-new-logo.png",
            "alt": "Microsoft Azure"
          },
          {
            "src": "https://assets.ubuntu.com/v1/a554a818-google-cloud-logo.png",
            "alt": "Google Cloud Platform"
          },
          {
            "src": "https://assets.ubuntu.com/v1/b3e692f4-oracle-new-logo.png",
            "alt": "Oracle"
          }
        ]
      }
    }
  ]
) -%}
{%- if slot == 'description' -%}
<p>The quick brown fox jumps over the lazy dog</p>
{%- endif -%}
{% endcall -%}
```

Notes
- Use `logo-block` for simple logo lists; `cta-block` to add buttons/links.
- The macro applies section padding automatically; you can override with `padding`.

---

## Tab section

Purpose: organize related content into separate tabs within a section with title, optional description, and CTA.

Key points
- Required params: `title` (Object with `text`), `tabs` (Array of tab objects).
- Layouts: `'full-width'`, `'50-50'` (default), `'25-75'`.
- Each tab has `type` (content block type) and `item` (block config).
- Supported block types vary by layout (e.g., quote only in full-width).
- JavaScript required for tab interactivity.

Jinja import
```/dev/null/tab-section-import.jinja#L1-3
{% from "_macros/vf_tab-section.jinja" import vf_tab_section %}
```

Minimal usage
```/dev/null/tab-section-example.jinja#L1-40
{% from "_macros/vf_tab-section.jinja" import vf_tab_section %}

{{ vf_tab_section(
  title={"text": "Features"},
  description={"content": "Explore our key features", "type": "text"},
  layout="50-50",
  tabs=[
    {
      "tab_html": "Logos",
      "type": "logo-block",
      "item": {
        "logos": [
          {"attrs": {"src": "https://assets.ubuntu.com/v1/cd5f636a-hp-logo.png", "alt": "HP"}},
          {"attrs": {"src": "https://assets.ubuntu.com/v1/f90702cd-lenovo-logo.png", "alt": "Lenovo"}}
        ]
      }
    },
    {
      "tab_html": "Blog",
      "type": "blog",
      "item": {
        "articles": [
          {
            "title": {"text": "Getting started", "link_attrs": {"href": "#"}},
            "description": {"text": "Learn the basics"},
            "metadata": {
              "authors": [{"text": "Author Name", "link_attrs": {"href": "#"}}],
              "date": {"text": "15 March 2025"}
            }
          }
        ]
      }
    }
  ]
) }}
```

Notes
- Block types: `quote`, `linked-logo`, `logo-block`, `divided-section`, `blog`, `basic-section`.
- Full-width supports quote; 50/50 supports divided-section and basic-section; all support linked-logo, logo-block, blog.
- Requires JS module: `import {tabs} from 'vanilla-framework/js'; tabs.initTabs('[role="tablist"]');`

---

## Tiered list

Purpose: list of paired titles and descriptions with optional top-level description and CTAs.

Key points
- Required params: `is_description_full_width_on_desktop` (bool), `is_list_full_width_on_tablet` (bool).
- Uses slots: `title`, `description` (optional), `list_item_title_[1-25]`, `list_item_description_[1-25]`, `cta` (optional).
- Layouts determined by boolean flags (50/50 vs full-width on different breakpoints).
- Max 25 list items.

Jinja import
```/dev/null/tiered-list-import.jinja#L1-3
{% from "_macros/vf_tiered-list.jinja" import vf_tiered_list %}
```

Minimal usage
```/dev/null/tiered-list-example.jinja#L1-30
{% from "_macros/vf_tiered-list.jinja" import vf_tiered_list %}

{% call(slot) vf_tiered_list(
  is_description_full_width_on_desktop=true,
  is_list_full_width_on_tablet=false
) %}
  {% if slot == 'title' %}
    <h2>Key benefits</h2>
  {% elif slot == 'description' %}
    <p>Discover what makes our solution unique.</p>
  {% elif slot == 'list_item_title_1' %}
    <h3>Fast deployment</h3>
  {% elif slot == 'list_item_description_1' %}
    <p>Get up and running in minutes with our streamlined setup.</p>
  {% elif slot == 'list_item_title_2' %}
    <h3>Enterprise support</h3>
  {% elif slot == 'list_item_description_2' %}
    <p>24/7 support from our expert team.</p>
  {% elif slot == 'cta' %}
    <a href="#" class="p-button--positive">Get started</a>
  {% endif %}
{% endcall %}
```

Notes
- `is_description_full_width_on_desktop=true` makes title/description span full width on desktop.
- `is_list_full_width_on_tablet=false` makes list items display side-by-side on tablet.
- Use numbered slots (`list_item_title_1`, `list_item_description_1`, etc.) for each list item pair.

---

## Basic section

Purpose: flexible 2-column (default) content section composed of various content blocks (description, images, lists, code, logos, CTAs).

Key points
- Required param: `title` (Object with `text` and optional `link_attrs`).
- Common params: `label_text`, `subtitle`, `items` (Array of block objects), `is_split_on_medium`, `padding`, `top_rule_variant`.
- Blocks support `type` keys: `description`, `image`, `video`, `list`, `code-block`, `logo-block`, `cta-block`, `notification`, etc.

Jinja import
```/dev/null/basic-section-import.jinja#L1-3
{% from "_macros/vf_basic-section.jinja" import vf_basic_section %}
```

Minimal usage (mixed items)
```/dev/null/basic-section-example.jinja#L1-28
{% from "_macros/vf_basic-section.jinja" import vf_basic_section %}

{% set items = [
  {'type':'description','item':{'type':'text','content':'Intro paragraph.'}},
  {'type':'image','item':{'attrs':{'src':'/img/feature.jpg','alt':'Feature'},'aspect_ratio':'3-2'}},
  {'type':'cta-block','item':{'primary':{'content_html':'Try demo','attrs':{'href':'/demo'}}}}
] %}

{{ vf_basic_section(title={'text':'Section title'}, items=items, is_split_on_medium=true) }}
```

Notes
- Use `is_split_on_medium=true` to split layout on tablet and larger.
- Items are rendered in sequence; each item supports its own options (see macros for details).
- Import full Vanilla SCSS for images/utility classes used by blocks.

---

General notes
- Always import the appropriate macro from `_macros/*.jinja`.
- Patterns rely on Vanilla CSS utilities — recommended to import the full framework or required partials in your project SCSS.
- When a pattern provides named slots (callable blocks), use `{% call(slotname) %}...{% endcall %}` to inject markup.
- Keep content structure consistent across repeated items to maintain visual rhythm.


---

# Suggestions Data

The following is the JSON array of location-grouped suggestions to implement.
Process each location one by one, applying all suggestions for that location before moving to the next.

```json
[
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 1,
        "table_id": "table-1",
        "table_title": "",
        "row_index": 3,
        "column_index": 2,
        "column_header": "",
        "row_header": "Page description (160 characters max)"
      },
      "in_metadata": true
    },
    "suggestions": [
      {
        "id": "suggest.kdc2v8qsypte",
        "anchor": {
          "preceding_text": "a\n\nPage title (60 characters max)\nFull-stack AI infrastructure\nPage description (160 characters max)\nFast-track your AI ",
          "following_text": " with a proven and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your model"
        },
        "change": {
          "type": "replace",
          "original_text": "journey",
          "new_text": "projects"
        },
        "verification": {
          "text_before_change": "a\n\nPage title (60 characters max)\nFull-stack AI infrastructure\nPage description (160 characters max)\nFast-track your AI journey with a proven and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your model",
          "text_after_change": "a\n\nPage title (60 characters max)\nFull-stack AI infrastructure\nPage description (160 characters max)\nFast-track your AI projects with a proven and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your model"
        },
        "position": {
          "start_index": 139,
          "end_index": 154
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "projects"
          },
          {
            "type": "delete",
            "original_text": "journey"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.1ubwnxu5p2c4",
        "anchor": {
          "preceding_text": " and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in ",
          "following_text": " infrastructuredata centre and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Su"
        },
        "change": {
          "type": "replace",
          "original_text": "the",
          "new_text": "your private"
        },
        "verification": {
          "text_before_change": " and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in the infrastructuredata centre and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Su",
          "text_after_change": " and supported open source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in your private infrastructuredata centre and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Su"
        },
        "position": {
          "start_index": 288,
          "end_index": 303
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "your private"
          },
          {
            "type": "delete",
            "original_text": "the"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.jvcl8y8mzzmq",
        "anchor": {
          "preceding_text": "pen source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in your privatethe ",
          "following_text": " and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Suggested navigation\nCanonic"
        },
        "change": {
          "type": "replace",
          "original_text": "data centre",
          "new_text": "infrastructure"
        },
        "verification": {
          "text_before_change": "pen source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in your privatethe data centre and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Suggested navigation\nCanonic",
          "text_after_change": "pen source AI infrastructure stack. Start quickly in the public cloud, then run your models at scale in your privatethe infrastructure and at the edge.\nSuggested page URL\nhttps://canonical.com/solutions/ai/infrastructure\nNEW *Suggested navigation\nCanonic"
        },
        "position": {
          "start_index": 304,
          "end_index": 329
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "infrastructure"
          },
          {
            "type": "delete",
            "original_text": "data centre"
          }
        ],
        "atomic_count": 2
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 2,
        "table_id": "table-2",
        "table_title": "TEMPLATE HERO",
        "row_index": 1,
        "column_index": 1,
        "column_header": "Build Yyour full stack for AI infrastructure at fo...",
        "row_header": "Build Yyour full stack for AI infrastructure at fo..."
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.r3eqy31u1iac",
        "anchor": {
          "preceding_text": "g\nPage type (choose from drop down)\n\nStage in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\n",
          "following_text": "our full stack for AI infrastructure at for scale\n\nFrom silicon to software. From development to production. Get all the"
        },
        "change": {
          "type": "replace",
          "original_text": "Y",
          "new_text": "Build y"
        },
        "verification": {
          "text_before_change": "g\nPage type (choose from drop down)\n\nStage in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack for AI infrastructure at for scale\n\nFrom silicon to software. From development to production. Get all the",
          "text_after_change": "g\nPage type (choose from drop down)\n\nStage in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild your full stack for AI infrastructure at for scale\n\nFrom silicon to software. From development to production. Get all the"
        },
        "position": {
          "start_index": 797,
          "end_index": 805
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Build "
          },
          {
            "type": "delete",
            "original_text": "Y"
          },
          {
            "type": "insert",
            "new_text": "y"
          }
        ],
        "atomic_count": 3
      },
      {
        "id": "suggest.bwehjghkbf2p",
        "anchor": {
          "preceding_text": " funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour ",
          "following_text": "AI infrastructure at for scale\n\nFrom silicon to software. From development to pr"
        },
        "change": {
          "type": "delete",
          "original_text": "full stack for "
        },
        "verification": {
          "text_before_change": " funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure at for scale\n\nFrom silicon to software. From development to pr",
          "text_after_change": " funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour AI infrastructure at for scale\n\nFrom silicon to software. From development to pr"
        },
        "position": {
          "start_index": 809,
          "end_index": 824
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "full stack for "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.w9p439nqlirv",
        "anchor": {
          "preceding_text": " in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure ",
          "following_text": "scale\n\nFrom silicon to software. From development to production. Get all the open source components you need to build yo"
        },
        "change": {
          "type": "replace",
          "original_text": "at ",
          "new_text": "for "
        },
        "verification": {
          "text_before_change": " in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure at scale\n\nFrom silicon to software. From development to production. Get all the open source components you need to build yo",
          "text_after_change": " in the funnel\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure for scale\n\nFrom silicon to software. From development to production. Get all the open source components you need to build yo"
        },
        "position": {
          "start_index": 842,
          "end_index": 849
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "at "
          },
          {
            "type": "insert",
            "new_text": "for "
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.rey8mf60xjoh",
        "anchor": {
          "preceding_text": "\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure at for scale\n\n",
          "following_text": "\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware ena"
        },
        "change": {
          "type": "replace",
          "original_text": "\n\nWe make AI infrastructure easy so you can focus on your models. Get all the infrastructure components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge.",
          "new_text": "From silicon to software. From development to production. Get all the open source components you need to build your AI infrastructure. Go to market quickly with an integrated end-to-end stack and start extracting value from your data at scale.\n\n\n"
        },
        "verification": {
          "text_before_change": "\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure at for scale\n\n\n\nWe make AI infrastructure easy so you can focus on your models. Get all the infrastructure components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge.\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware ena",
          "text_after_change": "\nExploration\n\n\n—Visible page starts here —\nTEMPLATE HERO\nBuild Yyour full stack for AI infrastructure at for scale\n\nFrom silicon to software. From development to production. Get all the open source components you need to build your AI infrastructure. Go to market quickly with an integrated end-to-end stack and start extracting value from your data at scale.\n\n\n\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware ena"
        },
        "position": {
          "start_index": 856,
          "end_index": 1304
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "From silicon to software. From development to production. Get all the open source components you need to build your AI infrastructure. Go to market quickly with an integrated end-to-end stack and start extracting value from your data at scale.\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "We make AI infrastructure easy so you can focus on your models. Get all the infrastructure components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge."
          }
        ],
        "atomic_count": 6
      },
      {
        "id": "suggest.iyyubqsdp5h9",
        "anchor": {
          "preceding_text": "cture components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge.\n\n[",
          "following_text": "]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end"
        },
        "change": {
          "type": "replace",
          "original_text": "Contact Canonical",
          "new_text": "Get in touch"
        },
        "verification": {
          "text_before_change": "cture components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge.\n\n[Contact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end",
          "text_after_change": "cture components you need for AI, from the operating system to the MLOps platform and all the way to the secure edge.\n\n[Get in touch]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end"
        },
        "position": {
          "start_index": 1307,
          "end_index": 1336
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Get in touch"
          },
          {
            "type": "delete",
            "original_text": "Contact Canonical"
          }
        ],
        "atomic_count": 2
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 3,
        "table_id": "table-3",
        "table_title": "TEMPLATE SPOTLIGHT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.iqmftym0p6za",
        "anchor": {
          "preceding_text": " the secure edge.\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\n",
          "following_text": "optimizsed for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with pre"
        },
        "change": {
          "type": "replace",
          "original_text": "Stable and supported end-to-end stack ",
          "new_text": "Deep hardware enablement "
        },
        "verification": {
          "text_before_change": " the secure edge.\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nStable and supported end-to-end stack optimizsed for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with pre",
          "text_after_change": " the secure edge.\n\n[Get in touchContact Canonical]\n\nTEMPLATE SPOTLIGHT\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement optimizsed for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with pre"
        },
        "position": {
          "start_index": 1412,
          "end_index": 1475
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Deep hardware enablement "
          },
          {
            "type": "delete",
            "original_text": "Stable and supported end-to-end stack "
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.ua09f1bf8478",
        "anchor": {
          "preceding_text": "T\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimi",
          "following_text": "ed for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with predictable"
        },
        "change": {
          "type": "replace",
          "original_text": "s",
          "new_text": "z"
        },
        "verification": {
          "text_before_change": "T\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimised for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with predictable",
          "text_after_change": "T\nWhy Canonical for enterprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimized for AI performance\nCost-effective model with one subscription for th  e entire stackControl your TCO with predictable"
        },
        "position": {
          "start_index": 1481,
          "end_index": 1483
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "z"
          },
          {
            "type": "delete",
            "original_text": "s"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.krz699r5ij17",
        "anchor": {
          "preceding_text": "rprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimizsed for AI performance\n",
          "following_text": "\n100% open source with enterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compl"
        },
        "change": {
          "type": "replace",
          "original_text": "Control your TCO with predictable pricing per node",
          "new_text": "Cost-effective model with one subscription for th  e entire stack\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments"
        },
        "verification": {
          "text_before_change": "rprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimizsed for AI performance\nControl your TCO with predictable pricing per node\n100% open source with enterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compl",
          "text_after_change": "rprise AI infrastructure?\n\nDeep hardware enablement Stable and supported end-to-end stack optimizsed for AI performance\nCost-effective model with one subscription for th  e entire stack\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments\n100% open source with enterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compl"
        },
        "position": {
          "start_index": 1505,
          "end_index": 1704
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Cost-effective model with one subscription for th  e entire stack"
          },
          {
            "type": "delete",
            "original_text": "Control your TCO with predictable pricing per node"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "Run "
          },
          {
            "type": "insert",
            "new_text": "your workloads "
          },
          {
            "type": "insert",
            "new_text": "anywhere – on premises, in public clouds, or hybrid environments"
          }
        ],
        "atomic_count": 6
      },
      {
        "id": "suggest.5l6mycfw8tjt",
        "anchor": {
          "preceding_text": "on for th  e entire stackControl your TCO with predictable pricing per node\nRun ",
          "following_text": "anywhere – on premises, in public clouds, or hybrid environments\n100% open sou"
        },
        "change": {
          "type": "insert",
          "new_text": "your workloads "
        },
        "verification": {
          "text_before_change": "on for th  e entire stackControl your TCO with predictable pricing per node\nRun anywhere – on premises, in public clouds, or hybrid environments\n100% open sou",
          "text_after_change": "on for th  e entire stackControl your TCO with predictable pricing per node\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments\n100% open sou"
        },
        "position": {
          "start_index": 1625,
          "end_index": 1640
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "your workloads "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ng9f2talxqia",
        "anchor": {
          "preceding_text": " with predictable pricing per node\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments",
          "following_text": "\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compliance with security across all layers of "
        },
        "change": {
          "type": "insert",
          "new_text": "\n100% open source with enterprise support"
        },
        "verification": {
          "text_before_change": " with predictable pricing per node\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compliance with security across all layers of ",
          "text_after_change": " with predictable pricing per node\nRun your workloads anywhere – on premises, in public clouds, or hybrid environments\n100% open source with enterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compliance with security across all layers of "
        },
        "position": {
          "start_index": 1704,
          "end_index": 1745
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "100% open source with enterprise support"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.die0tb4iqse",
        "anchor": {
          "preceding_text": " public clouds, or hybrid environments\n100% open source with enterprise support\n",
          "following_text": "\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEI"
        },
        "change": {
          "type": "delete",
          "original_text": "Design and deploy your AI infrastructure with expert guidance"
        },
        "verification": {
          "text_before_change": " public clouds, or hybrid environments\n100% open source with enterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEI",
          "text_after_change": " public clouds, or hybrid environments\n100% open source with enterprise support\n\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEI"
        },
        "position": {
          "start_index": 1746,
          "end_index": 1807
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Design and deploy your AI infrastructure with expert guidance"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.aqzkjfdzp2ge",
        "anchor": {
          "preceding_text": "nterprise support\nDesign and deploy your AI infrastructure with expert guidance\n",
          "following_text": "\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagra"
        },
        "change": {
          "type": "delete",
          "original_text": "Fast-track compliance with security across all layers of the stack"
        },
        "verification": {
          "text_before_change": "nterprise support\nDesign and deploy your AI infrastructure with expert guidance\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagra",
          "text_after_change": "nterprise support\nDesign and deploy your AI infrastructure with expert guidance\n\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagra"
        },
        "position": {
          "start_index": 1808,
          "end_index": 1874
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Fast-track compliance with security across all layers of the stack"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " guidance\nFast-track compliance with security across all layers of the stack\n\n\n\n",
          "following_text": "Full-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stac"
        },
        "change": {
          "type": "insert",
          "new_text": "EQUAL HEIGHT ROW\n"
        },
        "verification": {
          "text_before_change": " guidance\nFast-track compliance with security across all layers of the stack\n\n\n\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stac",
          "text_after_change": " guidance\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stac"
        },
        "position": {
          "start_index": 1882,
          "end_index": 1899
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "EQUAL HEIGHT ROW\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 4,
        "table_id": "table-4",
        "table_title": "EQUAL HEIGHT ROW",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ack compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\n",
          "following_text": "\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting "
        },
        "change": {
          "type": "insert",
          "new_text": "Full-stack AI infrastructure\n"
        },
        "verification": {
          "text_before_change": "ack compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting ",
          "text_after_change": "ack compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting "
        },
        "position": {
          "start_index": 1902,
          "end_index": 1931
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Full-stack AI infrastructure\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n",
          "following_text": "\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting w"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting w",
          "text_after_change": "across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting w"
        },
        "position": {
          "start_index": 1931,
          "end_index": 1932
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "cross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n",
          "following_text": "Alt: Technology stack diagram depicting full-stack AI infrastructure starting wi"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "cross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting wi",
          "text_after_change": "cross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting wi"
        },
        "position": {
          "start_index": 1934,
          "end_index": 1935
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ncf9tygdoavf",
        "anchor": {
          "preceding_text": "e\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\n",
          "following_text": "\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPU"
        },
        "change": {
          "type": "insert",
          "new_text": "Alt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM."
        },
        "verification": {
          "text_before_change": "e\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\n\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPU",
          "text_after_change": "e\nFast-track compliance with security across all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPU"
        },
        "position": {
          "start_index": 1935,
          "end_index": 2494
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Alt: Technology stack diagram"
          },
          {
            "type": "insert",
            "new_text": " depicting full-stack AI infrastructure"
          },
          {
            "type": "insert",
            "new_text": " starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM."
          }
        ],
        "atomic_count": 3
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\n",
          "following_text": " depicting full-stack AI infrastructure starting with silicon partners Intel, NV"
        },
        "change": {
          "type": "insert",
          "new_text": "Alt: Technology stack diagram"
        },
        "verification": {
          "text_before_change": "ross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\n depicting full-stack AI infrastructure starting with silicon partners Intel, NV",
          "text_after_change": "ross all layers of the stack\n\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NV"
        },
        "position": {
          "start_index": 1935,
          "end_index": 1964
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Alt: Technology stack diagram"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.y4omzx2oybu6",
        "anchor": {
          "preceding_text": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram",
          "following_text": " starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and"
        },
        "change": {
          "type": "insert",
          "new_text": " depicting full-stack AI infrastructure"
        },
        "verification": {
          "text_before_change": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and",
          "text_after_change": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and"
        },
        "position": {
          "start_index": 1964,
          "end_index": 2003
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " depicting full-stack AI infrastructure"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram",
          "following_text": " starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and"
        },
        "change": {
          "type": "insert",
          "new_text": " depicting full-stack AI infrastructure"
        },
        "verification": {
          "text_before_change": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and",
          "text_after_change": "\n\n\nEQUAL HEIGHT ROW\nFull-stack AI infrastructure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and"
        },
        "position": {
          "start_index": 1964,
          "end_index": 2003
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " depicting full-stack AI infrastructure"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "structure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure",
          "following_text": "\n\nHardware automation\n\nAutomate the provisioning and management of your data cen"
        },
        "change": {
          "type": "insert",
          "new_text": " starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM."
        },
        "verification": {
          "text_before_change": "structure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure\n\nHardware automation\n\nAutomate the provisioning and management of your data cen",
          "text_after_change": "structure\n\n\nAlt: Technology stack diagram depicting full-stack AI infrastructure starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data cen"
        },
        "position": {
          "start_index": 2003,
          "end_index": 2494
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " starting with silicon partners Intel, NVIDIA, AMD, MediaTek, Arm, Qualcomm, and Ampere. Above this, Canonical MAAS handles hardware automation and Canonical Ubuntu serves as the operating system. The infrastructure management layer displays Canonical OpenStack, Kubernetes, Landscape, and MicroCloud. Data solutions include PostgreSQL, Cassandra, Apache Spark, etcd, MySQL, MongoDB, Redis, Kafka, Ceph, OpenSearch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "rch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.",
          "following_text": "\nHardware automation\n\nAutomate the provisioning and management of your data cent"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "rch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\nHardware automation\n\nAutomate the provisioning and management of your data cent",
          "text_after_change": "rch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data cent"
        },
        "position": {
          "start_index": 2494,
          "end_index": 2495
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ak4dks8l90lq",
        "anchor": {
          "preceding_text": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n",
          "following_text": "Hardware automation\n\nAutomate the provisioning and management of your data cente"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\nHardware automation\n\nAutomate the provisioning and management of your data cente",
          "text_after_change": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data cente"
        },
        "position": {
          "start_index": 2495,
          "end_index": 2496
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n",
          "following_text": "Hardware automation\n\nAutomate the provisioning and management of your data cente"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\nHardware automation\n\nAutomate the provisioning and management of your data cente",
          "text_after_change": "ch, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data cente"
        },
        "position": {
          "start_index": 2495,
          "end_index": 2496
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "h, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\n",
          "following_text": "\n\nAutomate the provisioning and management of your data center hardware at scale"
        },
        "change": {
          "type": "insert",
          "new_text": "Hardware automation"
        },
        "verification": {
          "text_before_change": "h, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\n\n\nAutomate the provisioning and management of your data center hardware at scale",
          "text_after_change": "h, and Valkey. The top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale"
        },
        "position": {
          "start_index": 2496,
          "end_index": 2515
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Hardware automation"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation",
          "following_text": "\nAutomate the provisioning and management of your data center hardware at scale,"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\nAutomate the provisioning and management of your data center hardware at scale,",
          "text_after_change": "top AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale,"
        },
        "position": {
          "start_index": 2515,
          "end_index": 2516
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "op AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n",
          "following_text": "Automate the provisioning and management of your data center hardware at scale, "
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "op AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\nAutomate the provisioning and management of your data center hardware at scale, ",
          "text_after_change": "op AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale, "
        },
        "position": {
          "start_index": 2516,
          "end_index": 2517
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "p AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\n",
          "following_text": " at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI wor"
        },
        "change": {
          "type": "insert",
          "new_text": "Automate the provisioning and management of your data center hardware"
        },
        "verification": {
          "text_before_change": "p AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\n at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI wor",
          "text_after_change": "p AI/ML layer features Kubeflow, MLflow, Feast, and vLLM.\n\nHardware automation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI wor"
        },
        "position": {
          "start_index": 2517,
          "end_index": 2586
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Automate the provisioning and management of your data center hardware"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "utomation\n\nAutomate the provisioning and management of your data center hardware",
          "following_text": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n"
        },
        "change": {
          "type": "insert",
          "new_text": " at scale"
        },
        "verification": {
          "text_before_change": "utomation\n\nAutomate the provisioning and management of your data center hardware, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n",
          "text_after_change": "utomation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n"
        },
        "position": {
          "start_index": 2586,
          "end_index": 2595
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " at scale"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.mcu5c16asw1t",
        "anchor": {
          "preceding_text": "utomation\n\nAutomate the provisioning and management of your data center hardware",
          "following_text": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n"
        },
        "change": {
          "type": "delete",
          "original_text": " at scale"
        },
        "verification": {
          "text_before_change": "utomation\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n",
          "text_after_change": "utomation\n\nAutomate the provisioning and management of your data center hardware, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n"
        },
        "position": {
          "start_index": 2586,
          "end_index": 2595
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": " at scale"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "\n\nAutomate the provisioning and management of your data center hardware at scale",
          "following_text": "\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Ge"
        },
        "change": {
          "type": "insert",
          "new_text": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads."
        },
        "verification": {
          "text_before_change": "\n\nAutomate the provisioning and management of your data center hardware at scale\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Ge",
          "text_after_change": "\n\nAutomate the provisioning and management of your data center hardware at scale, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Ge"
        },
        "position": {
          "start_index": 2595,
          "end_index": 2673
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "le, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.",
          "following_text": "\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "le, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get",
          "text_after_change": "le, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get"
        },
        "position": {
          "start_index": 2673,
          "end_index": 2674
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "e, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n",
          "following_text": "Operating system\n\nUbuntu is the de facto operating system of choice for AI. Get "
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "e, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get ",
          "text_after_change": "e, including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get "
        },
        "position": {
          "start_index": 2674,
          "end_index": 2675
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n",
          "following_text": "\nUbuntu is the de facto operating system of choice for AI. Get access to the bro"
        },
        "change": {
          "type": "insert",
          "new_text": "Operating system\n"
        },
        "verification": {
          "text_before_change": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\n\nUbuntu is the de facto operating system of choice for AI. Get access to the bro",
          "text_after_change": ", including the GPUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the bro"
        },
        "position": {
          "start_index": 2675,
          "end_index": 2692
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Operating system\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "PUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n",
          "following_text": "Ubuntu is the de facto operating system of choice for AI. Get access to the broa"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "PUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\nUbuntu is the de facto operating system of choice for AI. Get access to the broa",
          "text_after_change": "PUs, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broa"
        },
        "position": {
          "start_index": 2692,
          "end_index": 2693
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "Us, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\n",
          "following_text": "de facto operating system of choice for AI. Get access to the broadest ecosystem"
        },
        "change": {
          "type": "insert",
          "new_text": "Ubuntu is the "
        },
        "verification": {
          "text_before_change": "Us, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nde facto operating system of choice for AI. Get access to the broadest ecosystem",
          "text_after_change": "Us, SmartNICs, and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem"
        },
        "position": {
          "start_index": 2693,
          "end_index": 2707
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Ubuntu is the "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.7tohz635t303",
        "anchor": {
          "preceding_text": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the ",
          "following_text": "operating system of choice for AI. Get access to the broadest ecosystem of machi"
        },
        "change": {
          "type": "delete",
          "original_text": "de facto "
        },
        "verification": {
          "text_before_change": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machi",
          "text_after_change": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the operating system of choice for AI. Get access to the broadest ecosystem of machi"
        },
        "position": {
          "start_index": 2707,
          "end_index": 2716
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "de facto "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the ",
          "following_text": "operating system of choice for AI. Get access to the broadest ecosystem of machi"
        },
        "change": {
          "type": "insert",
          "new_text": "de facto "
        },
        "verification": {
          "text_before_change": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the operating system of choice for AI. Get access to the broadest ecosystem of machi",
          "text_after_change": " and DPUs you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machi"
        },
        "position": {
          "start_index": 2707,
          "end_index": 2716
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "de facto "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto ",
          "following_text": "of choice for AI. Get access to the broadest ecosystem of machine learning tools"
        },
        "change": {
          "type": "insert",
          "new_text": "operating system "
        },
        "verification": {
          "text_before_change": " you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto of choice for AI. Get access to the broadest ecosystem of machine learning tools",
          "text_after_change": " you need to drive your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machine learning tools"
        },
        "position": {
          "start_index": 2716,
          "end_index": 2733
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "operating system "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.fww57kg87uz9",
        "anchor": {
          "preceding_text": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system ",
          "following_text": "for AI. Get access to the broadest ecosystem of machine learning tools and libra"
        },
        "change": {
          "type": "insert",
          "new_text": "of choice "
        },
        "verification": {
          "text_before_change": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system for AI. Get access to the broadest ecosystem of machine learning tools and libra",
          "text_after_change": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machine learning tools and libra"
        },
        "position": {
          "start_index": 2733,
          "end_index": 2743
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "of choice "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system ",
          "following_text": "for AI. Get access to the broadest ecosystem of machine learning tools and libra"
        },
        "change": {
          "type": "insert",
          "new_text": "of choice "
        },
        "verification": {
          "text_before_change": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system for AI. Get access to the broadest ecosystem of machine learning tools and libra",
          "text_after_change": "e your AI workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machine learning tools and libra"
        },
        "position": {
          "start_index": 2733,
          "end_index": 2743
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "of choice "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice ",
          "following_text": " the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modern"
        },
        "change": {
          "type": "insert",
          "new_text": "for AI. Get access to the broadest ecosystem of machine learning tools and libraries, and diversify your silicon supply chain for resiliency with an OS optimized for all"
        },
        "verification": {
          "text_before_change": "workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice  the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modern",
          "text_after_change": "workloads.\n\nOperating system\n\nUbuntu is the de facto operating system of choice for AI. Get access to the broadest ecosystem of machine learning tools and libraries, and diversify your silicon supply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modern"
        },
        "position": {
          "start_index": 2743,
          "end_index": 2912
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "for AI. Get access to the broadest ecosystem of machine learning tools and libraries, and diversify your silicon supply chain for resiliency with an OS optimized for all"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " diversify your silicon supply chain for resiliency with an OS optimized for all",
          "following_text": " leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize "
        },
        "change": {
          "type": "insert",
          "new_text": " the"
        },
        "verification": {
          "text_before_change": " diversify your silicon supply chain for resiliency with an OS optimized for all leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize ",
          "text_after_change": " diversify your silicon supply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize "
        },
        "position": {
          "start_index": 2912,
          "end_index": 2916
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " the"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.l58wrhos1wl0",
        "anchor": {
          "preceding_text": " diversify your silicon supply chain for resiliency with an OS optimized for all",
          "following_text": " leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize "
        },
        "change": {
          "type": "insert",
          "new_text": " the"
        },
        "verification": {
          "text_before_change": " diversify your silicon supply chain for resiliency with an OS optimized for all leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize ",
          "text_after_change": " diversify your silicon supply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize "
        },
        "position": {
          "start_index": 2912,
          "end_index": 2916
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " the"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ersify your silicon supply chain for resiliency with an OS optimized for all the",
          "following_text": "architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your work"
        },
        "change": {
          "type": "insert",
          "new_text": " leading "
        },
        "verification": {
          "text_before_change": "ersify your silicon supply chain for resiliency with an OS optimized for all thearchitectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your work",
          "text_after_change": "ersify your silicon supply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your work"
        },
        "position": {
          "start_index": 2916,
          "end_index": 2925
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " leading "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ur silicon supply chain for resiliency with an OS optimized for all the leading ",
          "following_text": ".\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI,"
        },
        "change": {
          "type": "insert",
          "new_text": "architectures"
        },
        "verification": {
          "text_before_change": "ur silicon supply chain for resiliency with an OS optimized for all the leading .\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI,",
          "text_after_change": "ur silicon supply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI,"
        },
        "position": {
          "start_index": 2925,
          "end_index": 2938
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "architectures"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "pply chain for resiliency with an OS optimized for all the leading architectures",
          "following_text": "\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, "
        },
        "change": {
          "type": "insert",
          "new_text": "."
        },
        "verification": {
          "text_before_change": "pply chain for resiliency with an OS optimized for all the leading architectures\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, ",
          "text_after_change": "pply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, "
        },
        "position": {
          "start_index": 2938,
          "end_index": 2939
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ply chain for resiliency with an OS optimized for all the leading architectures.",
          "following_text": "\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, m"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ply chain for resiliency with an OS optimized for all the leading architectures.\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, m",
          "text_after_change": "ply chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, m"
        },
        "position": {
          "start_index": 2939,
          "end_index": 2940
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ly chain for resiliency with an OS optimized for all the leading architectures.\n",
          "following_text": "Infrastructure management\n\nBuild your cloud, modernize your workloads for AI, ma"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ly chain for resiliency with an OS optimized for all the leading architectures.\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, ma",
          "text_after_change": "ly chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, ma"
        },
        "position": {
          "start_index": 2940,
          "end_index": 2941
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "y chain for resiliency with an OS optimized for all the leading architectures.\n\n",
          "following_text": "\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu es"
        },
        "change": {
          "type": "insert",
          "new_text": "Infrastructure management\n"
        },
        "verification": {
          "text_before_change": "y chain for resiliency with an OS optimized for all the leading architectures.\n\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu es",
          "text_after_change": "y chain for resiliency with an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu es"
        },
        "position": {
          "start_index": 2941,
          "end_index": 2967
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Infrastructure management\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "h an OS optimized for all the leading architectures.\n\nInfrastructure management\n",
          "following_text": "Build your cloud, modernize your workloads for AI, manage your entire Ubuntu est"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "h an OS optimized for all the leading architectures.\n\nInfrastructure management\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu est",
          "text_after_change": "h an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu est"
        },
        "position": {
          "start_index": 2967,
          "end_index": 2968
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " an OS optimized for all the leading architectures.\n\nInfrastructure management\n\n",
          "following_text": "DeployEnjoy bottom-up infrastructure automation so you can focus on your busines"
        },
        "change": {
          "type": "insert",
          "new_text": "Build your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. "
        },
        "verification": {
          "text_before_change": " an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nDeployEnjoy bottom-up infrastructure automation so you can focus on your busines",
          "text_after_change": " an OS optimized for all the leading architectures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your busines"
        },
        "position": {
          "start_index": 2968,
          "end_index": 3053
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Build your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.5zbybsx6ldqw",
        "anchor": {
          "preceding_text": "tures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. ",
          "following_text": " bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without"
        },
        "change": {
          "type": "replace",
          "original_text": "Enjoy",
          "new_text": "Deploy"
        },
        "verification": {
          "text_before_change": "tures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. Enjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without",
          "text_after_change": "tures.\n\nInfrastructure management\n\nBuild your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. Deploy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without"
        },
        "position": {
          "start_index": 3053,
          "end_index": 3064
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Deploy"
          },
          {
            "type": "delete",
            "original_text": "Enjoy"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. ",
          "following_text": "Enjoy bottom-up infrastructure automation so you can focus on your business.mode"
        },
        "change": {
          "type": "insert",
          "new_text": "Deploy"
        },
        "verification": {
          "text_before_change": " your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. Enjoy bottom-up infrastructure automation so you can focus on your business.mode",
          "text_after_change": " your cloud, modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.mode"
        },
        "position": {
          "start_index": 3053,
          "end_index": 3059
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Deploy"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "cloud, modernize your workloads for AI, manage your entire Ubuntu estate. Deploy",
          "following_text": " bottom-up infrastructure automation so you can focus on your business.models. \n"
        },
        "change": {
          "type": "insert",
          "new_text": "Enjoy"
        },
        "verification": {
          "text_before_change": "cloud, modernize your workloads for AI, manage your entire Ubuntu estate. Deploy bottom-up infrastructure automation so you can focus on your business.models. \n",
          "text_after_change": "cloud, modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n"
        },
        "position": {
          "start_index": 3059,
          "end_index": 3064
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Enjoy"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": ", modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy",
          "following_text": "infrastructure automation so you can focus on your business.models. \n\nData solut"
        },
        "change": {
          "type": "insert",
          "new_text": " bottom-up "
        },
        "verification": {
          "text_before_change": ", modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoyinfrastructure automation so you can focus on your business.models. \n\nData solut",
          "text_after_change": ", modernize your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solut"
        },
        "position": {
          "start_index": 3064,
          "end_index": 3075
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " bottom-up "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.f1i226akjmgy",
        "anchor": {
          "preceding_text": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up ",
          "following_text": "automation so you can focus on your business.models. \n\nData solutions\n\nThere’s"
        },
        "change": {
          "type": "insert",
          "new_text": "infrastructure "
        },
        "verification": {
          "text_before_change": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up automation so you can focus on your business.models. \n\nData solutions\n\nThere’s",
          "text_after_change": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s"
        },
        "position": {
          "start_index": 3075,
          "end_index": 3090
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "infrastructure "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up ",
          "following_text": "automation so you can focus on your business.models. \n\nData solutions\n\nThere’s"
        },
        "change": {
          "type": "insert",
          "new_text": "infrastructure "
        },
        "verification": {
          "text_before_change": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up automation so you can focus on your business.models. \n\nData solutions\n\nThere’s",
          "text_after_change": " your workloads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s"
        },
        "position": {
          "start_index": 3075,
          "end_index": 3090
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "infrastructure "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure ",
          "following_text": "business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical deliv"
        },
        "change": {
          "type": "insert",
          "new_text": "automation so you can focus on your "
        },
        "verification": {
          "text_before_change": " for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical deliv",
          "text_after_change": " for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical deliv"
        },
        "position": {
          "start_index": 3090,
          "end_index": 3126
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "automation so you can focus on your "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.p2bttf7qwp96",
        "anchor": {
          "preceding_text": "oads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your ",
          "following_text": ". \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source"
        },
        "change": {
          "type": "replace",
          "original_text": "models",
          "new_text": "business."
        },
        "verification": {
          "text_before_change": "oads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source",
          "text_after_change": "oads for AI, manage your entire Ubuntu estate. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source"
        },
        "position": {
          "start_index": 3126,
          "end_index": 3141
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "business."
          },
          {
            "type": "delete",
            "original_text": "models"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "state. DeployEnjoy bottom-up infrastructure automation so you can focus on your ",
          "following_text": "models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a com"
        },
        "change": {
          "type": "insert",
          "new_text": "business."
        },
        "verification": {
          "text_before_change": "state. DeployEnjoy bottom-up infrastructure automation so you can focus on your models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a com",
          "text_after_change": "state. DeployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a com"
        },
        "position": {
          "start_index": 3126,
          "end_index": 3135
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "business."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ployEnjoy bottom-up infrastructure automation so you can focus on your business.",
          "following_text": ". \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehen"
        },
        "change": {
          "type": "insert",
          "new_text": "models"
        },
        "verification": {
          "text_before_change": "ployEnjoy bottom-up infrastructure automation so you can focus on your business.. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehen",
          "text_after_change": "ployEnjoy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehen"
        },
        "position": {
          "start_index": 3135,
          "end_index": 3141
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "models"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "joy bottom-up infrastructure automation so you can focus on your business.models",
          "following_text": "\n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensi"
        },
        "change": {
          "type": "insert",
          "new_text": ". "
        },
        "verification": {
          "text_before_change": "joy bottom-up infrastructure automation so you can focus on your business.models\n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensi",
          "text_after_change": "joy bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensi"
        },
        "position": {
          "start_index": 3141,
          "end_index": 3143
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": ". "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "y bottom-up infrastructure automation so you can focus on your business.models. ",
          "following_text": "\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensiv"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "y bottom-up infrastructure automation so you can focus on your business.models. \nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensiv",
          "text_after_change": "y bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensiv"
        },
        "position": {
          "start_index": 3143,
          "end_index": 3144
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " bottom-up infrastructure automation so you can focus on your business.models. \n",
          "following_text": "Data solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": " bottom-up infrastructure automation so you can focus on your business.models. \nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive",
          "text_after_change": " bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive"
        },
        "position": {
          "start_index": 3144,
          "end_index": 3145
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "bottom-up infrastructure automation so you can focus on your business.models. \n\n",
          "following_text": "\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of i"
        },
        "change": {
          "type": "insert",
          "new_text": "Data solutions\n"
        },
        "verification": {
          "text_before_change": "bottom-up infrastructure automation so you can focus on your business.models. \n\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of i",
          "text_after_change": "bottom-up infrastructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of i"
        },
        "position": {
          "start_index": 3145,
          "end_index": 3160
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Data solutions\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "structure automation so you can focus on your business.models. \n\nData solutions\n",
          "following_text": "There’s no AI without data. Canonical delivers a comprehensive portfolio of in"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "structure automation so you can focus on your business.models. \n\nData solutions\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of in",
          "text_after_change": "structure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of in"
        },
        "position": {
          "start_index": 3160,
          "end_index": 3161
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "tructure automation so you can focus on your business.models. \n\nData solutions\n\n",
          "following_text": "\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to "
        },
        "change": {
          "type": "insert",
          "new_text": "There’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source solutions for database management, data processing, search, and caching."
        },
        "verification": {
          "text_before_change": "tructure automation so you can focus on your business.models. \n\nData solutions\n\n\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to ",
          "text_after_change": "tructure automation so you can focus on your business.models. \n\nData solutions\n\nThere’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source solutions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to "
        },
        "position": {
          "start_index": 3161,
          "end_index": 3332
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "There’s no AI without data. Canonical delivers a comprehensive portfolio of integrated open source solutions for database management, data processing, search, and caching."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " source solutions for database management, data processing, search, and caching.",
          "following_text": "\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to b"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": " source solutions for database management, data processing, search, and caching.\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to b",
          "text_after_change": " source solutions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to b"
        },
        "position": {
          "start_index": 3332,
          "end_index": 3333
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "source solutions for database management, data processing, search, and caching.\n",
          "following_text": "MLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to br"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "source solutions for database management, data processing, search, and caching.\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to br",
          "text_after_change": "source solutions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to br"
        },
        "position": {
          "start_index": 3333,
          "end_index": 3334
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.foqe3zr48j30",
        "anchor": {
          "preceding_text": "rehensive portfolio of integrated open source solutions for database management, data processing, search, and caching.\n\n",
          "following_text": "\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation t"
        },
        "change": {
          "type": "replace",
          "original_text": "AI/ML",
          "new_text": "MLOps"
        },
        "verification": {
          "text_before_change": "rehensive portfolio of integrated open source solutions for database management, data processing, search, and caching.\n\nAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation t",
          "text_after_change": "rehensive portfolio of integrated open source solutions for database management, data processing, search, and caching.\n\nMLOps\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation t"
        },
        "position": {
          "start_index": 3334,
          "end_index": 3344
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "MLOps"
          },
          {
            "type": "delete",
            "original_text": "AI/ML"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ource solutions for database management, data processing, search, and caching.\n\n",
          "following_text": "AI/ML\n\nOur modular MLOps platform equips you with everything you need to bring y"
        },
        "change": {
          "type": "insert",
          "new_text": "MLOps"
        },
        "verification": {
          "text_before_change": "ource solutions for database management, data processing, search, and caching.\n\nAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring y",
          "text_after_change": "ource solutions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring y"
        },
        "position": {
          "start_index": 3334,
          "end_index": 3339
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "MLOps"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " solutions for database management, data processing, search, and caching.\n\nMLOps",
          "following_text": "\n\nOur modular MLOps platform equips you with everything you need to bring your m"
        },
        "change": {
          "type": "insert",
          "new_text": "AI/ML"
        },
        "verification": {
          "text_before_change": " solutions for database management, data processing, search, and caching.\n\nMLOps\n\nOur modular MLOps platform equips you with everything you need to bring your m",
          "text_after_change": " solutions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring your m"
        },
        "position": {
          "start_index": 3339,
          "end_index": 3344
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "AI/ML"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "tions for database management, data processing, search, and caching.\n\nMLOpsAI/ML",
          "following_text": "\nOur modular MLOps platform equips you with everything you need to bring your mo"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "tions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\nOur modular MLOps platform equips you with everything you need to bring your mo",
          "text_after_change": "tions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring your mo"
        },
        "position": {
          "start_index": 3344,
          "end_index": 3345
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n",
          "following_text": "Our modular MLOps platform equips you with everything you need to bring your mod"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\nOur modular MLOps platform equips you with everything you need to bring your mod",
          "text_after_change": "ions for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring your mod"
        },
        "position": {
          "start_index": 3345,
          "end_index": 3346
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ons for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\n",
          "following_text": "\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the"
        },
        "change": {
          "type": "insert",
          "new_text": "Our modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production."
        },
        "verification": {
          "text_before_change": "ons for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\n\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the",
          "text_after_change": "ons for database management, data processing, search, and caching.\n\nMLOpsAI/ML\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the"
        },
        "position": {
          "start_index": 3346,
          "end_index": 3477
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Our modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "ng you need to bring your models all the way from experimentation to production.",
          "following_text": "\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the "
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ng you need to bring your models all the way from experimentation to production.\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the ",
          "text_after_change": "ng you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the "
        },
        "position": {
          "start_index": 3477,
          "end_index": 3478
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "g you need to bring your models all the way from experimentation to production.\n",
          "following_text": "LOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the c"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "g you need to bring your models all the way from experimentation to production.\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the c",
          "text_after_change": "g you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the c"
        },
        "position": {
          "start_index": 3479,
          "end_index": 3480
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": false,
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " you need to bring your models all the way from experimentation to production.\n\n",
          "following_text": "Optimized for leading silicons\nTreat the chaotic layer of Solve the challenge of"
        },
        "change": {
          "type": "insert",
          "new_text": "LOGO CLOUD\n"
        },
        "verification": {
          "text_before_change": " you need to bring your models all the way from experimentation to production.\n\nOptimized for leading silicons\nTreat the chaotic layer of Solve the challenge of",
          "text_after_change": " you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the challenge of"
        },
        "position": {
          "start_index": 3480,
          "end_index": 3491
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "LOGO CLOUD\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "D logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\n",
          "following_text": "Tested and certified with the world’s largest hardware vendors\nDon’t get bot"
        },
        "change": {
          "type": "insert",
          "new_text": "LOGO CLOUD\n"
        },
        "verification": {
          "text_before_change": "D logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nTested and certified with the world’s largest hardware vendors\nDon’t get bot",
          "text_after_change": "D logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bot"
        },
        "position": {
          "start_index": 3797,
          "end_index": 3808
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "LOGO CLOUD\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": " [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n",
          "following_text": "TEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, publi"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": " [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, publi",
          "text_after_change": " [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, publi"
        },
        "position": {
          "start_index": 4116,
          "end_index": 4117
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.p1lw1pey6ow2",
        "anchor": {
          "preceding_text": "vironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\n",
          "following_text": "Trusted across clouds\nGet started easily on your cloud of choice with optimized "
        },
        "change": {
          "type": "insert",
          "new_text": "LOGO CLOUD\n"
        },
        "verification": {
          "text_before_change": "vironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nTrusted across clouds\nGet started easily on your cloud of choice with optimized ",
          "text_after_change": "vironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with optimized "
        },
        "position": {
          "start_index": 4695,
          "end_index": 4706
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "LOGO CLOUD\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.p1lw1pey6ow2",
        "anchor": {
          "preceding_text": "images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n",
          "following_text": "\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optiona"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optiona",
          "text_after_change": "images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optiona"
        },
        "position": {
          "start_index": 4891,
          "end_index": 4892
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.aqud8mklpeuw",
        "anchor": {
          "preceding_text": "ages. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\n",
          "following_text": "Deploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "ages. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time",
          "text_after_change": "ages. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time"
        },
        "position": {
          "start_index": 4893,
          "end_index": 4913
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "rney, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\n",
          "following_text": "Stay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes inv"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "rney, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nTEMPLATE BASIC TEXT\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes inv",
          "text_after_change": "rney, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes inv"
        },
        "position": {
          "start_index": 5232,
          "end_index": 5252
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "e pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\n",
          "following_text": "\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical te"
        },
        "change": {
          "type": "delete",
          "original_text": "What customers say\n"
        },
        "verification": {
          "text_before_change": "e pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical te",
          "text_after_change": "e pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical te"
        },
        "position": {
          "start_index": 5692,
          "end_index": 5711
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "What customers say\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 5,
        "table_id": "table-5",
        "table_title": "LOGO CLOUD",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "rm equips you with everything you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\n",
          "following_text": "LOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level wo"
        },
        "change": {
          "type": "insert",
          "new_text": "Optimized for leading silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement as a solved problem with AI infrastructure solutions that run optimally across all major architectures.\n\n[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\n"
        },
        "verification": {
          "text_before_change": "rm equips you with everything you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level wo",
          "text_after_change": "rm equips you with everything you need to bring your models all the way from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement as a solved problem with AI infrastructure solutions that run optimally across all major architectures.\n\n[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level wo"
        },
        "position": {
          "start_index": 3494,
          "end_index": 3797
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Optimized for leading silicons\n"
          },
          {
            "type": "insert",
            "new_text": "Treat the chaotic layer of"
          },
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "insert",
            "new_text": "Solve the challenge of "
          },
          {
            "type": "insert",
            "new_text": "silicon enablement "
          },
          {
            "type": "insert",
            "new_text": "as a solved problem "
          },
          {
            "type": "insert",
            "new_text": "with AI infrastructure solutions that run optimally across all major architectures.\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 10
      },
      {
        "id": "suggest.mpn9l04iqob",
        "anchor": {
          "preceding_text": " from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\n",
          "following_text": " Solve the challenge of silicon enablement as a solved problem with AI infrastru"
        },
        "change": {
          "type": "delete",
          "original_text": "Treat the chaotic layer of"
        },
        "verification": {
          "text_before_change": " from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement as a solved problem with AI infrastru",
          "text_after_change": " from experimentation to production.\n\nLOGO CLOUD\nOptimized for leading silicons\n Solve the challenge of silicon enablement as a solved problem with AI infrastru"
        },
        "position": {
          "start_index": 3525,
          "end_index": 3551
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Treat the chaotic layer of"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.496xngtve8fe",
        "anchor": {
          "preceding_text": "oduction.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of ",
          "following_text": "silicon enablement as a solved problem with AI infrastructure solutions that run"
        },
        "change": {
          "type": "insert",
          "new_text": "Solve the challenge of "
        },
        "verification": {
          "text_before_change": "oduction.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of silicon enablement as a solved problem with AI infrastructure solutions that run",
          "text_after_change": "oduction.\n\nLOGO CLOUD\nOptimized for leading silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement as a solved problem with AI infrastructure solutions that run"
        },
        "position": {
          "start_index": 3552,
          "end_index": 3575
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Solve the challenge of "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.139iqhxjaew5",
        "anchor": {
          "preceding_text": "g silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement ",
          "following_text": "with AI infrastructure solutions that run optimally across all major architectur"
        },
        "change": {
          "type": "delete",
          "original_text": "as a solved problem "
        },
        "verification": {
          "text_before_change": "g silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement as a solved problem with AI infrastructure solutions that run optimally across all major architectur",
          "text_after_change": "g silicons\nTreat the chaotic layer of Solve the challenge of silicon enablement with AI infrastructure solutions that run optimally across all major architectur"
        },
        "position": {
          "start_index": 3594,
          "end_index": 3614
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "as a solved problem "
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 6,
        "table_id": "table-6",
        "table_title": "LOGO CLOUD",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.ua0eu3c7146n",
        "anchor": {
          "preceding_text": "itectures.\n\n[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nLOGO CLOUD\n",
          "following_text": "\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid – deploy anyw"
        },
        "change": {
          "type": "insert",
          "new_text": "Tested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. Accelerate time-to-value with validated hardware. \n\n[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n"
        },
        "verification": {
          "text_before_change": "itectures.\n\n[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nLOGO CLOUD\n\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid – deploy anyw",
          "text_after_change": "itectures.\n\n[NVIDIA Logo] [AMD logo] [Intel logo] [Qualcomm logo] [Mediatek logo] [arm logo] [Ampere logo] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. Accelerate time-to-value with validated hardware. \n\n[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid – deploy anyw"
        },
        "position": {
          "start_index": 3811,
          "end_index": 4116
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Tested and certified with the world’s largest hardware vendors\n"
          },
          {
            "type": "insert",
            "new_text": "Don’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. "
          },
          {
            "type": "insert",
            "new_text": "Accelerate time-to-value with validated hardware. \n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 6
      },
      {
        "id": "suggest.d06rbrc9ljf6",
        "anchor": {
          "preceding_text": "] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\n",
          "following_text": "Accelerate time-to-value with validated hardware. \n\n[Dell logo] [HPE logo] [hp l"
        },
        "change": {
          "type": "delete",
          "original_text": "Don’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. "
        },
        "verification": {
          "text_before_change": "] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nDon’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. Accelerate time-to-value with validated hardware. \n\n[Dell logo] [HPE logo] [hp l",
          "text_after_change": "] \n\nLOGO CLOUD\nTested and certified with the world’s largest hardware vendors\nAccelerate time-to-value with validated hardware. \n\n[Dell logo] [HPE logo] [hp l"
        },
        "position": {
          "start_index": 3874,
          "end_index": 3972
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Don’t get bottlenecked by the low-level work required to ensure hardware will work in your fleet. "
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 7,
        "table_id": "table-7",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.f64hkgxjx6c0",
        "anchor": {
          "preceding_text": "ware. \n\n[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, start quickly with no risk and low investme"
        },
        "change": {
          "type": "replace",
          "original_text": "Deploy anywhere – private, public, or hybrid",
          "new_text": "Private, public, sovereign, or hybrid – deploy anywhere"
        },
        "verification": {
          "text_before_change": "ware. \n\n[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n\nTEMPLATE BASIC TEXT\nDeploy anywhere – private, public, or hybrid\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, start quickly with no risk and low investme",
          "text_after_change": "ware. \n\n[Dell logo] [HPE logo] [hp logo] [Huawei logo] [IBM logo] [Lenovo logo] [Supermicro logo]\n\n\nTEMPLATE BASIC TEXT\nPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, start quickly with no risk and low investme"
        },
        "position": {
          "start_index": 4140,
          "end_index": 4239
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Deploy anywhere – private, public, or hybrid"
          },
          {
            "type": "insert",
            "new_text": "Private, public, "
          },
          {
            "type": "insert",
            "new_text": "sovereign, "
          },
          {
            "type": "insert",
            "new_text": "or hybrid "
          },
          {
            "type": "insert",
            "new_text": "– deploy anywhere"
          }
        ],
        "atomic_count": 5
      },
      {
        "id": "suggest.9ayfbog70cri",
        "anchor": {
          "preceding_text": "PLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, ",
          "following_text": "or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use "
        },
        "change": {
          "type": "insert",
          "new_text": "sovereign, "
        },
        "verification": {
          "text_before_change": "PLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use ",
          "text_after_change": "PLATE BASIC TEXT\nDeploy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use "
        },
        "position": {
          "start_index": 4201,
          "end_index": 4212
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "sovereign, "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.icqd2cdrps7e",
        "anchor": {
          "preceding_text": "oy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid ",
          "following_text": "\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, sta"
        },
        "change": {
          "type": "delete",
          "original_text": "– deploy anywhere"
        },
        "verification": {
          "text_before_change": "oy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, sta",
          "text_after_change": "oy anywhere – private, public, or hybridPrivate, public, sovereign, or hybrid \n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, sta"
        },
        "position": {
          "start_index": 4222,
          "end_index": 4239
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "– deploy anywhere"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ky98shsr20ha",
        "anchor": {
          "preceding_text": "brid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases",
          "following_text": " – fFor instance, start quickly with no risk and low investment on public clou"
        },
        "change": {
          "type": "insert",
          "new_text": "."
        },
        "verification": {
          "text_before_change": "brid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases – fFor instance, start quickly with no risk and low investment on public clou",
          "text_after_change": "brid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. – fFor instance, start quickly with no risk and low investment on public clou"
        },
        "position": {
          "start_index": 4296,
          "end_index": 4297
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.8cj1f8b0rrpn",
        "anchor": {
          "preceding_text": "ybridPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. ",
          "following_text": "or instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to "
        },
        "change": {
          "type": "replace",
          "original_text": "– f",
          "new_text": "F"
        },
        "verification": {
          "text_before_change": "ybridPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. – for instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to ",
          "text_after_change": "ybridPrivate, public, sovereign, or hybrid – deploy anywhere\n\n\nChoose the ideal AI infrastructure for your use cases. For instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to "
        },
        "position": {
          "start_index": 4298,
          "end_index": 4302
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "– f"
          },
          {
            "type": "insert",
            "new_text": "F"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.b8ldb7snd9tk",
        "anchor": {
          "preceding_text": "rastructure for your use cases. – fFor instance, start quickly with no risk and low investment on public clouds, then ",
          "following_text": " private cloud and migrate workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run"
        },
        "change": {
          "type": "replace",
          "original_text": "move to a",
          "new_text": "migrate your workloads to your"
        },
        "verification": {
          "text_before_change": "rastructure for your use cases. – fFor instance, start quickly with no risk and low investment on public clouds, then move to a private cloud and migrate workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run",
          "text_after_change": "rastructure for your use cases. – fFor instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to your private cloud and migrate workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run"
        },
        "position": {
          "start_index": 4384,
          "end_index": 4423
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "migrate your workloads to your"
          },
          {
            "type": "delete",
            "original_text": "move "
          },
          {
            "type": "delete",
            "original_text": "to a"
          }
        ],
        "atomic_count": 3
      },
      {
        "id": "suggest.os6vxh20o857",
        "anchor": {
          "preceding_text": " fFor instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove ",
          "following_text": "workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, i"
        },
        "change": {
          "type": "insert",
          "new_text": "to a private cloud and migrate "
        },
        "verification": {
          "text_before_change": " fFor instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, i",
          "text_after_change": " fFor instance, start quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to a private cloud and migrate workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, i"
        },
        "position": {
          "start_index": 4419,
          "end_index": 4450
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "to a"
          },
          {
            "type": "insert",
            "new_text": " private cloud "
          },
          {
            "type": "insert",
            "new_text": "and migrate "
          }
        ],
        "atomic_count": 3
      },
      {
        "id": "suggest.wsl566b6gle5",
        "anchor": {
          "preceding_text": "rt quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to a private cloud ",
          "following_text": "as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mult"
        },
        "change": {
          "type": "delete",
          "original_text": "and migrate workloads to your own data centreer "
        },
        "verification": {
          "text_before_change": "rt quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to a private cloud and migrate workloads to your own data centreer as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mult",
          "text_after_change": "rt quickly with no risk and low investment on public clouds, then migrate your workloads to yourmove to a private cloud as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mult"
        },
        "position": {
          "start_index": 4438,
          "end_index": 4486
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "and migrate "
          },
          {
            "type": "delete",
            "original_text": "workloads to your own data cent"
          },
          {
            "type": "delete",
            "original_text": "re"
          },
          {
            "type": "delete",
            "original_text": "er"
          },
          {
            "type": "delete",
            "original_text": " "
          }
        ],
        "atomic_count": 5
      },
      {
        "id": "suggest.j19tvmgcl527",
        "anchor": {
          "preceding_text": "on public clouds, then migrate your workloads to yourmove to a private cloud and migrate workloads to your own data cent",
          "following_text": " as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mul"
        },
        "change": {
          "type": "replace",
          "original_text": "re",
          "new_text": "er"
        },
        "verification": {
          "text_before_change": "on public clouds, then migrate your workloads to yourmove to a private cloud and migrate workloads to your own data centre as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mul",
          "text_after_change": "on public clouds, then migrate your workloads to yourmove to a private cloud and migrate workloads to your own data center as you scale. \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and mul"
        },
        "position": {
          "start_index": 4481,
          "end_index": 4485
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "re"
          },
          {
            "type": "insert",
            "new_text": "er"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.frb02ibdkdr9",
        "anchor": {
          "preceding_text": " \n\nWith Canonical’s solutions, you can run your workloads anywhere, including ",
          "following_text": "hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cl"
        },
        "change": {
          "type": "insert",
          "new_text": "sovereign, "
        },
        "verification": {
          "text_before_change": " \n\nWith Canonical’s solutions, you can run your workloads anywhere, including hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cl",
          "text_after_change": " \n\nWith Canonical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cl"
        },
        "position": {
          "start_index": 4577,
          "end_index": 4588
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "sovereign, "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.pijgiz6xyxi4",
        "anchor": {
          "preceding_text": "’s solutions, you can run your workloads anywhere, including sovereign, hybrid",
          "following_text": " and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud str"
        },
        "change": {
          "type": "insert",
          "new_text": ","
        },
        "verification": {
          "text_before_change": "’s solutions, you can run your workloads anywhere, including sovereign, hybrid and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud str",
          "text_after_change": "’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud str"
        },
        "position": {
          "start_index": 4594,
          "end_index": 4595
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": ","
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.iaxj1urblvae",
        "anchor": {
          "preceding_text": "onical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and multi-cloud environments. \n\n",
          "following_text": "\n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with optimized and certified Ubuntu images"
        },
        "change": {
          "type": "replace",
          "original_text": "Download our hybrid cloud strategy playbook \u003e ",
          "new_text": "Build your cloud \u003e"
        },
        "verification": {
          "text_before_change": "onical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and multi-cloud environments. \n\nDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with optimized and certified Ubuntu images",
          "text_after_change": "onical’s solutions, you can run your workloads anywhere, including sovereign, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003e\n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with optimized and certified Ubuntu images"
        },
        "position": {
          "start_index": 4628,
          "end_index": 4692
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Build your cloud"
          },
          {
            "type": "insert",
            "new_text": " \u003e"
          },
          {
            "type": "delete",
            "original_text": "Download our hybrid cloud strategy playbook \u003e "
          }
        ],
        "atomic_count": 3
      },
      {
        "id": "suggest.p1lw1pey6ow2",
        "anchor": {
          "preceding_text": "nvironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n",
          "following_text": "LOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "nvironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with",
          "text_after_change": "nvironments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with"
        },
        "position": {
          "start_index": 4694,
          "end_index": 4695
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 8,
        "table_id": "table-8",
        "table_title": "LOGO CLOUD",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.p1lw1pey6ow2",
        "anchor": {
          "preceding_text": "n, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\n",
          "following_text": "\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time data processing i"
        },
        "change": {
          "type": "insert",
          "new_text": "Trusted across clouds\nGet started easily on your cloud of choice with optimized and certified Ubuntu images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n"
        },
        "verification": {
          "text_before_change": "n, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time data processing i",
          "text_after_change": "n, hybrid, and multi-cloud environments. \n\nBuild your cloud \u003eDownload our hybrid cloud strategy playbook \u003e \n\nLOGO CLOUD\nTrusted across clouds\nGet started easily on your cloud of choice with optimized and certified Ubuntu images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time data processing i"
        },
        "position": {
          "start_index": 4709,
          "end_index": 4891
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Trusted across clouds\n"
          },
          {
            "type": "insert",
            "new_text": "Get started easily on your cloud of choice with optimized and certified Ubuntu images. "
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 6
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 9,
        "table_id": "table-9",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 1,
        "column_index": 2,
        "column_header": "[Image (optional)]",
        "row_header": "Deploy machine learning models to the edge"
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.aqud8mklpeuw",
        "anchor": {
          "preceding_text": " certified Ubuntu images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "\nTEMPLATE BASIC TEXT\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes involved in AI project"
        },
        "change": {
          "type": "delete",
          "original_text": "Deploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time data processing in distributed environments by deploying machine learning models to your edge devices. \n\nCanonical infrastructure spans the entire AI journey, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n"
        },
        "verification": {
          "text_before_change": " certified Ubuntu images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\nDeploy machine learning models to the edge\n\n[Image (optional)]\n\nUnlock real-time data processing in distributed environments by deploying machine learning models to your edge devices. \n\nCanonical infrastructure spans the entire AI journey, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nTEMPLATE BASIC TEXT\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes involved in AI project",
          "text_after_change": " certified Ubuntu images. \n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo]\n\n\n\nTEMPLATE BASIC TEXT\n\nTEMPLATE BASIC TEXT\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes involved in AI project"
        },
        "position": {
          "start_index": 4916,
          "end_index": 5230
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Deploy machine learning models to the edge\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "[Image (optional)]"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Unlock real-time data processing in distributed environments by deploying machine learning models to your edge devices. \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Canonical infrastructure spans the entire AI journey, from the desktop to the edge.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Access our guide to open source edge AI "
          },
          {
            "type": "delete",
            "original_text": "\u003e\n"
          }
        ],
        "atomic_count": 11
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "in_table": true,
      "table": {
        "table_index": 10,
        "table_id": "table-10",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "ns the entire AI journey, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "What customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical team was remarkable. Ev"
        },
        "change": {
          "type": "delete",
          "original_text": "Stay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes involved in AI projects can scale rapidly, making public cloud prohibitively costly. If operated efficiently, a private cloud is more cost-effective for running workloads long-term and at scale.\n\nBuild your private cloud with our AI infrastructure solutions and enjoy predictable pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\n"
        },
        "verification": {
          "text_before_change": "ns the entire AI journey, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nTEMPLATE BASIC TEXT\nStay in control of AI infrastructure costs\n\n[Image (optional)]\n\nData volumes involved in AI projects can scale rapidly, making public cloud prohibitively costly. If operated efficiently, a private cloud is more cost-effective for running workloads long-term and at scale.\n\nBuild your private cloud with our AI infrastructure solutions and enjoy predictable pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical team was remarkable. Ev",
          "text_after_change": "ns the entire AI journey, from the desktop to the edge.\n\nAccess our guide to open source edge AI \u003e\n\nTEMPLATE BASIC TEXT\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical team was remarkable. Ev"
        },
        "position": {
          "start_index": 5255,
          "end_index": 5692
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Stay in control of AI infrastructure costs\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "[Image (optional)]"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Data volumes involved in AI projects can scale rapidly, making public cloud prohibitively costly. If operated efficiently, a private cloud is more cost-effective for running workloads long-term and at scale.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Build your private cloud with our AI infrastructure solutions and enjoy predictable pricing per node with no licence fees.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Check out our cloud pricing report"
          },
          {
            "type": "delete",
            "original_text": " "
          },
          {
            "type": "delete",
            "original_text": "\u003e\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 13
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": false,
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n",
          "following_text": "TEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical tea"
        },
        "change": {
          "type": "delete",
          "original_text": "\n"
        },
        "verification": {
          "text_before_change": "with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical tea",
          "text_after_change": "with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical tea"
        },
        "position": {
          "start_index": 5711,
          "end_index": 5712
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "ith no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\n",
          "following_text": "[Firmus logo] “The level of engagement from the Canonical team was remarkable."
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE QUOTE 1\n"
        },
        "verification": {
          "text_before_change": "ith no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical team was remarkable.",
          "text_after_change": "ith no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\n[Firmus logo] “The level of engagement from the Canonical team was remarkable."
        },
        "position": {
          "start_index": 5712,
          "end_index": 5729
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE QUOTE 1\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n",
          "following_text": "TEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google "
        },
        "change": {
          "type": "delete",
          "original_text": "\n"
        },
        "verification": {
          "text_before_change": "and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google ",
          "text_after_change": "and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google "
        },
        "position": {
          "start_index": 6087,
          "end_index": 6088
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "nd Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\n",
          "following_text": "The #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM log"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "nd Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM log",
          "text_after_change": "nd Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM log"
        },
        "position": {
          "start_index": 6088,
          "end_index": 6108
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "th optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\n",
          "following_text": "Accelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu cer"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "th optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nTEMPLATE BASIC TEXT\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu cer",
          "text_after_change": "th optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu cer"
        },
        "position": {
          "start_index": 6554,
          "end_index": 6574
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ad the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\n",
          "following_text": "From experimentation to production with a modular MLOps platform\n\nMachine learni"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "ad the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\nTEMPLATE BASIC TEXT\nFrom experimentation to production with a modular MLOps platform\n\nMachine learni",
          "text_after_change": "ad the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\nFrom experimentation to production with a modular MLOps platform\n\nMachine learni"
        },
        "position": {
          "start_index": 7951,
          "end_index": 7971
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "o key tools, you can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\n",
          "following_text": "AI infrastructure in the real world, powered by Canonical\n[Space exploration] [F"
        },
        "change": {
          "type": "insert",
          "new_text": "TABBED SECTIONS\n"
        },
        "verification": {
          "text_before_change": "o key tools, you can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [F",
          "text_after_change": "o key tools, you can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [F"
        },
        "position": {
          "start_index": 8469,
          "end_index": 8485
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "TABBED SECTIONS\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ng engineer, entertainment technology company\n\n[Download the full case study]\n\n\n",
          "following_text": "Jumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nCost"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE CTA\n"
        },
        "verification": {
          "text_before_change": "ng engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nCost",
          "text_after_change": "ng engineer, entertainment technology company\n\n[Download the full case study]\n\n\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nCost"
        },
        "position": {
          "start_index": 10408,
          "end_index": 10421
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE CTA\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.20dbhfhlfl6w",
        "anchor": {
          "preceding_text": "\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT",
          "following_text": "Start your journey with a workshopUnblock your AI/ML initiatives with an MLOps w"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXTStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps w",
          "text_after_change": "\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps w"
        },
        "position": {
          "start_index": 11170,
          "end_index": 11171
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.51l7m8dlpm1d",
        "anchor": {
          "preceding_text": "hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\n",
          "following_text": "Confidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardwar"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE BASIC TEXT\n"
        },
        "verification": {
          "text_before_change": "hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardwar",
          "text_after_change": "hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardwar"
        },
        "position": {
          "start_index": 11680,
          "end_index": 11700
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE BASIC TEXT\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": "quired pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\n",
          "following_text": "Build your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI in"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE CTA\n"
        },
        "verification": {
          "text_before_change": "quired pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI in",
          "text_after_change": "quired pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI in"
        },
        "position": {
          "start_index": 12263,
          "end_index": 12276
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE CTA\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": " more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n",
          "following_text": "TEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity o"
        },
        "change": {
          "type": "delete",
          "original_text": "\n"
        },
        "verification": {
          "text_before_change": " more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity o",
          "text_after_change": " more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity o"
        },
        "position": {
          "start_index": 12315,
          "end_index": 12316
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": "more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\n",
          "following_text": "Open source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-tim"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE LEARN MORE FOOTER\n"
        },
        "verification": {
          "text_before_change": "more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-tim",
          "text_after_change": "more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-tim"
        },
        "position": {
          "start_index": 12316,
          "end_index": 12343
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE LEARN MORE FOOTER\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": "harmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\n",
          "following_text": "Send us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n\ufffd"
        },
        "change": {
          "type": "delete",
          "original_text": "TEMPLATE CTA\n"
        },
        "verification": {
          "text_before_change": "harmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n\ufffd",
          "text_after_change": "harmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n\ufffd"
        },
        "position": {
          "start_index": 13330,
          "end_index": 13343
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "TEMPLATE CTA\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": ". \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n",
          "following_text": "\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack f"
        },
        "change": {
          "type": "delete",
          "original_text": "\n"
        },
        "verification": {
          "text_before_change": ". \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack f",
          "text_after_change": ". \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack f"
        },
        "position": {
          "start_index": 13408,
          "end_index": 13409
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 11,
        "table_id": "table-11",
        "table_title": "TEMPLATE QUOTE 1",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "able pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n",
          "following_text": "\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n\n"
        },
        "change": {
          "type": "delete",
          "original_text": "[Firmus logo] “The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around\nOEMs and hardware choices. They were dedicated to our project from the get-go.” \n\nTim Rosenfield\nCEO and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n"
        },
        "verification": {
          "text_before_change": "able pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n[Firmus logo] “The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around\nOEMs and hardware choices. They were dedicated to our project from the get-go.” \n\nTim Rosenfield\nCEO and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n\n",
          "text_after_change": "able pricing per node with no licence fees.\n\nCheck out our cloud pricing report \u003e\n\nWhat customers say\n\nTEMPLATE QUOTE 1\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n\n"
        },
        "position": {
          "start_index": 5732,
          "end_index": 6087
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "[Firmus logo] "
          },
          {
            "type": "delete",
            "original_text": "“The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around\n"
          },
          {
            "type": "delete",
            "original_text": "OEMs and hardware choices. They were dedicated to our project from the get-go.”"
          },
          {
            "type": "delete",
            "original_text": " \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Tim Rosenfield\n"
          },
          {
            "type": "delete",
            "original_text": "CEO and Co-Founder\n"
          },
          {
            "type": "delete",
            "original_text": "Firmus\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Download the full case study"
          },
          {
            "type": "delete",
            "original_text": " "
          },
          {
            "type": "delete",
            "original_text": "\u003e"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "[Embed "
          },
          {
            "type": "delete",
            "original_text": "related video"
          },
          {
            "type": "delete",
            "original_text": "] "
          },
          {
            "type": "delete",
            "original_text": " "
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 20
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 12,
        "table_id": "table-12",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "Tim Rosenfield\nCEO and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "TEMPLATE BASIC TEXT\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu certified hardware from"
        },
        "change": {
          "type": "delete",
          "original_text": "The #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n\nThanks to its security, versatility and policy of regular updates, Ubuntu is the most popular operating system across public clouds. And the best part is: it's free. You pay only for the commercial support you need.\n\nGet started easily on your cloud of choice with optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\n"
        },
        "verification": {
          "text_before_change": "Tim Rosenfield\nCEO and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\nThe #1 Linux in the cloud\n\n\n[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n\nThanks to its security, versatility and policy of regular updates, Ubuntu is the most popular operating system across public clouds. And the best part is: it's free. You pay only for the commercial support you need.\n\nGet started easily on your cloud of choice with optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nTEMPLATE BASIC TEXT\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu certified hardware from",
          "text_after_change": "Tim Rosenfield\nCEO and Co-Founder\nFirmus\n\nDownload the full case study \u003e\n[Embed related video]  \n\n\n\nTEMPLATE BASIC TEXT\nTEMPLATE BASIC TEXT\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu certified hardware from"
        },
        "position": {
          "start_index": 6111,
          "end_index": 6554
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "The #1 Linux in the cloud\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "[AWS Logo] [Azure logo] [Google Cloud logo] [IBM logo] [Oracle logo] \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Thanks to its security, versatility and policy of regular updates, Ubuntu is the most popular operating system across public clouds. And the best part is: it's free. You pay only for the commercial support you need.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Get started easily on your cloud of choice with optimised and certified images. \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Learn more about Ubuntu on public cloud"
          },
          {
            "type": "delete",
            "original_text": " \u003e\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 12
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 13,
        "table_id": "table-13",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.kx22366eqa7p",
        "anchor": {
          "preceding_text": "r cloud of choice with optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "TEMPLATE CTA\nRead the executive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for A"
        },
        "change": {
          "type": "delete",
          "original_text": "Accelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu certified hardware from all major OEMs. Choosing validated platforms significantly reduces deployment time and costs for end-customers, in addition to improving reliability and providing bug-fixes faster.\n\nFor example, combined with NVIDIA DGX systems or NVIDIA-Certified systems, choosing an Ubuntu-certified platform provides secure, performant hardware that is guaranteed to deploy quickly and run smoothly.\n\nExplore certified hardware \u003e\n\n"
        },
        "verification": {
          "text_before_change": "r cloud of choice with optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nTEMPLATE BASIC TEXT\nAccelerate innovation with certified hardware\n\nGet peace of mind with Ubuntu certified hardware from all major OEMs. Choosing validated platforms significantly reduces deployment time and costs for end-customers, in addition to improving reliability and providing bug-fixes faster.\n\nFor example, combined with NVIDIA DGX systems or NVIDIA-Certified systems, choosing an Ubuntu-certified platform provides secure, performant hardware that is guaranteed to deploy quickly and run smoothly.\n\nExplore certified hardware \u003e\n\nTEMPLATE CTA\nRead the executive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for A",
          "text_after_change": "r cloud of choice with optimised and certified images. \n\nLearn more about Ubuntu on public cloud \u003e\n\nTEMPLATE BASIC TEXT\nTEMPLATE CTA\nRead the executive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for A"
        },
        "position": {
          "start_index": 6577,
          "end_index": 7099
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Accelerate innovation with certified hardware\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Get peace of mind with Ubuntu certified hardware from all major OEMs. Choosing validated platforms significantly reduces deployment time and costs for end-customers, in addition to improving reliability and providing bug-fixes faster.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "For example, combined with NVIDIA DGX systems or NVIDIA-Certified systems, choosing an Ubuntu-certified platform provides secure, performant hardware that is guaranteed to deploy quickly and run smoothly.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Explore certified hardware"
          },
          {
            "type": "delete",
            "original_text": " \u003e"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 10
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 14,
        "table_id": "table-14",
        "table_title": "TEMPLATE CTA",
        "row_index": 1,
        "column_index": 1,
        "column_header": "Read the executive’s guide to managed AI infrast...",
        "row_header": "Read the executive’s guide to managed AI infrast..."
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.w02f2rakjhxw",
        "anchor": {
          "preceding_text": "to deploy quickly and run smoothly.\n\nExplore certified hardware \u003e\n\nTEMPLATE CTA\n",
          "following_text": "\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\nDoing m"
        },
        "change": {
          "type": "delete",
          "original_text": "Read the executive’s guide to managed AI infrastructure"
        },
        "verification": {
          "text_before_change": "to deploy quickly and run smoothly.\n\nExplore certified hardware \u003e\n\nTEMPLATE CTA\nRead the executive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\nDoing m",
          "text_after_change": "to deploy quickly and run smoothly.\n\nExplore certified hardware \u003e\n\nTEMPLATE CTA\n\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\nDoing m"
        },
        "position": {
          "start_index": 7115,
          "end_index": 7170
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Read the executive’s guide to managed AI infrastructure"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 15,
        "table_id": "table-15",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 1,
        "column_index": 2,
        "column_header": "[Image (optional)]",
        "row_header": "Kubernetes optimized for AI"
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.bjlazlow6lqg",
        "anchor": {
          "preceding_text": "o managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n",
          "following_text": "\nDoing more with AI means modernizing your workloads. Modernized workloads mean "
        },
        "change": {
          "type": "delete",
          "original_text": "[Image (optional)]\n"
        },
        "verification": {
          "text_before_change": "o managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\nDoing more with AI means modernizing your workloads. Modernized workloads mean ",
          "text_after_change": "o managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n\nDoing more with AI means modernizing your workloads. Modernized workloads mean "
        },
        "position": {
          "start_index": 7228,
          "end_index": 7247
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "[Image (optional)]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.y4sq4fuoc3xd",
        "anchor": {
          "preceding_text": "xecutive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\n",
          "following_text": "\n\nCanonical Kubernetes is optimizsed to enhance AI/ML performance, incorporating features that amplify processing power "
        },
        "change": {
          "type": "replace",
          "original_text": "\nKubernetes plays a pivotal role in orchestrating AI applications. Canonical delivers easy-to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. ",
          "new_text": "Doing more with AI means modernizing your workloads. Modernized workloads mean containerization – and containers need Kubernetes.\n"
        },
        "verification": {
          "text_before_change": "xecutive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\n\nKubernetes plays a pivotal role in orchestrating AI applications. Canonical delivers easy-to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. \n\nCanonical Kubernetes is optimizsed to enhance AI/ML performance, incorporating features that amplify processing power ",
          "text_after_change": "xecutive’s guide to managed AI infrastructure\n\n\n\nTEMPLATE BASIC TEXT\nKubernetes optimized for AI\n\n[Image (optional)]\n\nDoing more with AI means modernizing your workloads. Modernized workloads mean containerization – and containers need Kubernetes.\n\n\nCanonical Kubernetes is optimizsed to enhance AI/ML performance, incorporating features that amplify processing power "
        },
        "position": {
          "start_index": 7248,
          "end_index": 7556
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Doing more with AI means modernizing your workloads. Mo"
          },
          {
            "type": "insert",
            "new_text": "dernized"
          },
          {
            "type": "insert",
            "new_text": " workloads mean containerization – and containers need Kubernetes."
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Kubernetes plays a pivotal role in orchestrating AI applications. Canonical delivers easy-to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. "
          }
        ],
        "atomic_count": 6
      },
      {
        "id": "suggest.8ulobdjm7fqb",
        "anchor": {
          "preceding_text": "to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. \n\nCanonical Kubernetes is optimi",
          "following_text": "ed to enhance AI/ML performance, incorporating features that amplify processing power and reduce latency, developed and "
        },
        "change": {
          "type": "replace",
          "original_text": "s",
          "new_text": "z"
        },
        "verification": {
          "text_before_change": "to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. \n\nCanonical Kubernetes is optimised to enhance AI/ML performance, incorporating features that amplify processing power and reduce latency, developed and ",
          "text_after_change": "to-use, CNCF conformant Kubernetes distributions for hybrid and multi-cloud operations. \n\nCanonical Kubernetes is optimized to enhance AI/ML performance, incorporating features that amplify processing power and reduce latency, developed and "
        },
        "position": {
          "start_index": 7588,
          "end_index": 7590
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "z"
          },
          {
            "type": "delete",
            "original_text": "s"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.iuvg1eydj5g7",
        "anchor": {
          "preceding_text": "ormance, incorporating features that amplify processing power and reduce latency",
          "following_text": " developed and integrated in a tight collaboration with NVIDIA. For instance, yo"
        },
        "change": {
          "type": "insert",
          "new_text": ","
        },
        "verification": {
          "text_before_change": "ormance, incorporating features that amplify processing power and reduce latency developed and integrated in a tight collaboration with NVIDIA. For instance, yo",
          "text_after_change": "ormance, incorporating features that amplify processing power and reduce latency, developed and integrated in a tight collaboration with NVIDIA. For instance, yo"
        },
        "position": {
          "start_index": 7694,
          "end_index": 7695
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": ","
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.mz7co88r7wkh",
        "anchor": {
          "preceding_text": "s that amplify processing power and reduce latency, developed and integrated in ",
          "following_text": "tight collaboration with NVIDIA. For instance, you can optimise hardware utilisa"
        },
        "change": {
          "type": "delete",
          "original_text": "a "
        },
        "verification": {
          "text_before_change": "s that amplify processing power and reduce latency, developed and integrated in a tight collaboration with NVIDIA. For instance, you can optimise hardware utilisa",
          "text_after_change": "s that amplify processing power and reduce latency, developed and integrated in tight collaboration with NVIDIA. For instance, you can optimise hardware utilisa"
        },
        "position": {
          "start_index": 7724,
          "end_index": 7726
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "a "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.1upjzchj9i7j",
        "anchor": {
          "preceding_text": " reduce latency, developed and integrated in a tight collaboration with NVIDIA. ",
          "following_text": "\n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX s"
        },
        "change": {
          "type": "delete",
          "original_text": "For instance, you can optimise hardware utilisation through NVIDIA operators and the Volcano scheduler."
        },
        "verification": {
          "text_before_change": " reduce latency, developed and integrated in a tight collaboration with NVIDIA. For instance, you can optimise hardware utilisation through NVIDIA operators and the Volcano scheduler.\n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX s",
          "text_after_change": " reduce latency, developed and integrated in a tight collaboration with NVIDIA. \n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX s"
        },
        "position": {
          "start_index": 7759,
          "end_index": 7862
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "For instance, you can optimise hardware utilisation through NVIDIA operators and the Volcano scheduler."
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 16,
        "table_id": "table-16",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "o scheduler.\n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "TABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Clou"
        },
        "change": {
          "type": "replace",
          "original_text": "From experimentation to production with a modular MLOps platform\n\nMachine learning operations (MLOps) is like DevOps for machine learning. It is a set of practices that automates machine learning workflows, ensuring scalability, portability and reproducibility.\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production. With easy access to key tools, you can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\n",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "o scheduler.\n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\nTEMPLATE BASIC TEXT\nFrom experimentation to production with a modular MLOps platform\n\nMachine learning operations (MLOps) is like DevOps for machine learning. It is a set of practices that automates machine learning workflows, ensuring scalability, portability and reproducibility.\n\nOur modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production. With easy access to key tools, you can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Clou",
          "text_after_change": "o scheduler.\n\nDownload the solution brief: Kubernetes by Canonical delivered on NVIDIA DGX systems\n\nTEMPLATE BASIC TEXT\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Clou"
        },
        "position": {
          "start_index": 7974,
          "end_index": 8469
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "From experimentation to production with a modular MLOps platform\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Machine learning operations (MLOps) is like DevOps for machine learning. It is a set of practices that automates machine learning workflows, ensuring scalability, portability and reproducibility.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Our modular MLOps platform equips you with everything you need to bring your models all the way from experimentation to production. With easy access to key tools, you can move quickly and at scale.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Discover our MLOps platform"
          },
          {
            "type": "delete",
            "original_text": " \u003e\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 10
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 17,
        "table_id": "table-17",
        "table_title": "TABBED SECTIONS",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nTABBED SECTIONS\n",
          "following_text": "\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainm"
        },
        "change": {
          "type": "insert",
          "new_text": "AI infrastructure in the real world, powered by Canonical"
        },
        "verification": {
          "text_before_change": " can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nTABBED SECTIONS\n\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainm",
          "text_after_change": " can move quickly and at scale.\n\nDiscover our MLOps platform \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainm"
        },
        "position": {
          "start_index": 8488,
          "end_index": 8545
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "AI infrastructure in the real world, powered by Canonical"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "orm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical",
          "following_text": "[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainme"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "orm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainme",
          "text_after_change": "orm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainme"
        },
        "position": {
          "start_index": 8545,
          "end_index": 8546
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "rm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n",
          "following_text": "and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complex"
        },
        "change": {
          "type": "insert",
          "new_text": "[Space exploration] [Financial services] [Cloud hosting] [Media "
        },
        "verification": {
          "text_before_change": "rm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\nand\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complex",
          "text_after_change": "rm \u003e\n\nTABBED SECTIONS\nAI infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complex"
        },
        "position": {
          "start_index": 8546,
          "end_index": 8610
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "[Space exploration] [Financial services] [Cloud hosting] [Media "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ed by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media ",
          "following_text": "\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity"
        },
        "change": {
          "type": "insert",
          "new_text": "and"
        },
        "verification": {
          "text_before_change": "ed by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media \u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity",
          "text_after_change": "ed by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity"
        },
        "position": {
          "start_index": 8610,
          "end_index": 8613
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "and"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.43wks7i8adbh",
        "anchor": {
          "preceding_text": " infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media ",
          "following_text": " entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted "
        },
        "change": {
          "type": "replace",
          "original_text": "\u0026",
          "new_text": "and"
        },
        "verification": {
          "text_before_change": " infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media \u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted ",
          "text_after_change": " infrastructure in the real world, powered by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted "
        },
        "position": {
          "start_index": 8610,
          "end_index": 8614
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "and"
          },
          {
            "type": "delete",
            "original_text": "\u0026"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and",
          "following_text": " entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity "
        },
        "change": {
          "type": "insert",
          "new_text": "\u0026"
        },
        "verification": {
          "text_before_change": "by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity ",
          "text_after_change": "by Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity "
        },
        "position": {
          "start_index": 8613,
          "end_index": 8614
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\u0026"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "y Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026",
          "following_text": "\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical spac"
        },
        "change": {
          "type": "insert",
          "new_text": " entertainment]\n"
        },
        "verification": {
          "text_before_change": "y Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical spac",
          "text_after_change": "y Canonical\n[Space exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical spac"
        },
        "position": {
          "start_index": 8614,
          "end_index": 8630
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " entertainment]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ce exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n",
          "following_text": "\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ce exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space",
          "text_after_change": "ce exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space"
        },
        "position": {
          "start_index": 8631,
          "end_index": 8632
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "e exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n",
          "following_text": "Space exploration\n[ESA logo]\nLowering the cost and complexity of critical space "
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "e exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space ",
          "text_after_change": "e exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space "
        },
        "position": {
          "start_index": 8633,
          "end_index": 8634
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\n",
          "following_text": "[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wan"
        },
        "change": {
          "type": "insert",
          "new_text": "Space exploration\n"
        },
        "verification": {
          "text_before_change": " exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wan",
          "text_after_change": " exploration] [Financial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wan"
        },
        "position": {
          "start_index": 8636,
          "end_index": 8654
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Space exploration\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ancial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n",
          "following_text": "Lowering the cost and complexity of critical space missions\n“We wanted one par"
        },
        "change": {
          "type": "insert",
          "new_text": "[ESA logo]\n"
        },
        "verification": {
          "text_before_change": "ancial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\nLowering the cost and complexity of critical space missions\n“We wanted one par",
          "text_after_change": "ancial services] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted one par"
        },
        "position": {
          "start_index": 8655,
          "end_index": 8666
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "[ESA logo]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ices] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\n",
          "following_text": "“We wanted one partner for the whole on-premise cloud because we’re not just"
        },
        "change": {
          "type": "insert",
          "new_text": "Lowering the cost and complexity of critical space missions\n"
        },
        "verification": {
          "text_before_change": "ices] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\n“We wanted one partner for the whole on-premise cloud because we’re not just",
          "text_after_change": "ices] [Cloud hosting] [Media and\u0026 entertainment]\n\n\nSpace exploration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted one partner for the whole on-premise cloud because we’re not just"
        },
        "position": {
          "start_index": 8666,
          "end_index": 8726
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Lowering the cost and complexity of critical space missions\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "loration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n",
          "following_text": "\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full c"
        },
        "change": {
          "type": "insert",
          "new_text": "“We wanted one partner for the whole on-premise cloud because we’re not just supporting Kubernetes but also our Ceph clusters, managed Postgres, Kafka, and AI tools such as Kubeflow and Spark. These were all the services that were needed and with this we could have one nice, easy joined-up approach.”\n"
        },
        "verification": {
          "text_before_change": "loration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full c",
          "text_after_change": "loration\n[ESA logo]\nLowering the cost and complexity of critical space missions\n“We wanted one partner for the whole on-premise cloud because we’re not just supporting Kubernetes but also our Ceph clusters, managed Postgres, Kafka, and AI tools such as Kubeflow and Spark. These were all the services that were needed and with this we could have one nice, easy joined-up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full c"
        },
        "position": {
          "start_index": 8726,
          "end_index": 9028
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "“We wanted one partner for the whole on-premise cloud because we’re not just supporting Kubernetes but also our Ceph clusters, managed Postgres, Kafka, and AI tools such as Kubeflow and Spark. These were all the services that were needed and with this we could have one nice, easy joined-up approach.”\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "t were needed and with this we could have one nice, easy joined-up approach.”\n",
          "following_text": "—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full ca"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "t were needed and with this we could have one nice, easy joined-up approach.”\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full ca",
          "text_after_change": "t were needed and with this we could have one nice, easy joined-up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full ca"
        },
        "position": {
          "start_index": 9028,
          "end_index": 9029
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " were needed and with this we could have one nice, easy joined-up approach.”\n\n",
          "following_text": "\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes "
        },
        "change": {
          "type": "insert",
          "new_text": "—Michael Hawkshaw, IT Service Manager, European Space Agency\n"
        },
        "verification": {
          "text_before_change": " were needed and with this we could have one nice, easy joined-up approach.”\n\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes ",
          "text_after_change": " were needed and with this we could have one nice, easy joined-up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes "
        },
        "position": {
          "start_index": 9029,
          "end_index": 9090
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "—Michael Hawkshaw, IT Service Manager, European Space Agency\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n",
          "following_text": "[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes f"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes f",
          "text_after_change": "up approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes f"
        },
        "position": {
          "start_index": 9090,
          "end_index": 9091
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "p approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n",
          "following_text": "Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes fr"
        },
        "change": {
          "type": "insert",
          "new_text": "["
        },
        "verification": {
          "text_before_change": "p approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\nGet the full case study]\n\nFinancial services\nFortune 500 fintech company goes fr",
          "text_after_change": "p approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes fr"
        },
        "position": {
          "start_index": 9091,
          "end_index": 9092
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "["
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[",
          "following_text": "]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshoo"
        },
        "change": {
          "type": "insert",
          "new_text": "Get the full case study"
        },
        "verification": {
          "text_before_change": " approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshoo",
          "text_after_change": " approach.”\n\n—Michael Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshoo"
        },
        "position": {
          "start_index": 9092,
          "end_index": 9115
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Get the full case study"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "el Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study",
          "following_text": "\nFinancial services\nFortune 500 fintech company goes from constant troubleshooti"
        },
        "change": {
          "type": "insert",
          "new_text": "]\n"
        },
        "verification": {
          "text_before_change": "el Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study\nFinancial services\nFortune 500 fintech company goes from constant troubleshooti",
          "text_after_change": "el Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshooti"
        },
        "position": {
          "start_index": 9115,
          "end_index": 9117
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n",
          "following_text": "Financial services\nFortune 500 fintech company goes from constant troubleshootin"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": " Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\nFinancial services\nFortune 500 fintech company goes from constant troubleshootin",
          "text_after_change": " Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshootin"
        },
        "position": {
          "start_index": 9118,
          "end_index": 9119
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\n",
          "following_text": "Fortune 500 fintech company goes from constant troubleshooting to scalable machi"
        },
        "change": {
          "type": "insert",
          "new_text": "Financial services\n"
        },
        "verification": {
          "text_before_change": "Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFortune 500 fintech company goes from constant troubleshooting to scalable machi",
          "text_after_change": "Hawkshaw, IT Service Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshooting to scalable machi"
        },
        "position": {
          "start_index": 9121,
          "end_index": 9140
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Financial services\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "e Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\n",
          "following_text": "“We like the whole integration umbrella that Canonical offers with Canonical K"
        },
        "change": {
          "type": "insert",
          "new_text": "Fortune 500 fintech company goes from constant troubleshooting to scalable machine learning\n"
        },
        "verification": {
          "text_before_change": "e Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\n“We like the whole integration umbrella that Canonical offers with Canonical K",
          "text_after_change": "e Manager, European Space Agency\n\n[Get the full case study]\n\nFinancial services\nFortune 500 fintech company goes from constant troubleshooting to scalable machine learning\n“We like the whole integration umbrella that Canonical offers with Canonical K"
        },
        "position": {
          "start_index": 9141,
          "end_index": 9233
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Fortune 500 fintech company goes from constant troubleshooting to scalable machine learning\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "fintech company goes from constant troubleshooting to scalable machine learning\n",
          "following_text": "\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full"
        },
        "change": {
          "type": "insert",
          "new_text": "“We like the whole integration umbrella that Canonical offers with Canonical Kubeflow. We have easy access to popular ML frameworks like TensorFlow, PyTorch, and XGBoost, as well as the native Kubernetes tools for monitoring and logging. It helps our data scientists quickly build and experiment with their models in an efficient manner.”\n"
        },
        "verification": {
          "text_before_change": "fintech company goes from constant troubleshooting to scalable machine learning\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full",
          "text_after_change": "fintech company goes from constant troubleshooting to scalable machine learning\n“We like the whole integration umbrella that Canonical offers with Canonical Kubeflow. We have easy access to popular ML frameworks like TensorFlow, PyTorch, and XGBoost, as well as the native Kubernetes tools for monitoring and logging. It helps our data scientists quickly build and experiment with their models in an efficient manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full"
        },
        "position": {
          "start_index": 9233,
          "end_index": 9572
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "“We like the whole integration umbrella that Canonical offers with Canonical Kubeflow. We have easy access to popular ML frameworks like TensorFlow, PyTorch, and XGBoost, as well as the native Kubernetes tools for monitoring and logging. It helps our data scientists quickly build and experiment with their models in an efficient manner.”\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "tists quickly build and experiment with their models in an efficient manner.”\n",
          "following_text": "—Senior data scientist, Fortune 500 financial services company\n\n[Get the full "
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "tists quickly build and experiment with their models in an efficient manner.”\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full ",
          "text_after_change": "tists quickly build and experiment with their models in an efficient manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full "
        },
        "position": {
          "start_index": 9572,
          "end_index": 9573
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ists quickly build and experiment with their models in an efficient manner.”\n\n",
          "following_text": "\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable c"
        },
        "change": {
          "type": "insert",
          "new_text": "—Senior data scientist, Fortune 500 financial services company\n"
        },
        "verification": {
          "text_before_change": "ists quickly build and experiment with their models in an efficient manner.”\n\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable c",
          "text_after_change": "ists quickly build and experiment with their models in an efficient manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable c"
        },
        "position": {
          "start_index": 9573,
          "end_index": 9636
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "—Senior data scientist, Fortune 500 financial services company\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "nt manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n",
          "following_text": "[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cl"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "nt manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cl",
          "text_after_change": "nt manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cl"
        },
        "position": {
          "start_index": 9636,
          "end_index": 9637
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "t manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n",
          "following_text": "Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable clo"
        },
        "change": {
          "type": "insert",
          "new_text": "["
        },
        "verification": {
          "text_before_change": "t manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\nGet the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable clo",
          "text_after_change": "t manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable clo"
        },
        "position": {
          "start_index": 9637,
          "end_index": 9638
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "["
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[",
          "following_text": "]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“"
        },
        "change": {
          "type": "insert",
          "new_text": "Get the full case study"
        },
        "verification": {
          "text_before_change": " manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“",
          "text_after_change": " manner.”\n\n—Senior data scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“"
        },
        "position": {
          "start_index": 9638,
          "end_index": 9661
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Get the full case study"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "data scientist, Fortune 500 financial services company\n\n[Get the full case study",
          "following_text": "\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“T"
        },
        "change": {
          "type": "insert",
          "new_text": "]"
        },
        "verification": {
          "text_before_change": "data scientist, Fortune 500 financial services company\n\n[Get the full case study\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“T",
          "text_after_change": "data scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“T"
        },
        "position": {
          "start_index": 9661,
          "end_index": 9662
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "]"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ata scientist, Fortune 500 financial services company\n\n[Get the full case study]",
          "following_text": "\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“Th"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ata scientist, Fortune 500 financial services company\n\n[Get the full case study]\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“Th",
          "text_after_change": "ata scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“Th"
        },
        "position": {
          "start_index": 9662,
          "end_index": 9663
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ta scientist, Fortune 500 financial services company\n\n[Get the full case study]\n",
          "following_text": "Cloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ta scientist, Fortune 500 financial services company\n\n[Get the full case study]\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The",
          "text_after_change": "ta scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The"
        },
        "position": {
          "start_index": 9664,
          "end_index": 9665
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "a scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\n",
          "following_text": "[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of enga"
        },
        "change": {
          "type": "insert",
          "new_text": "Cloud hosting\n"
        },
        "verification": {
          "text_before_change": "a scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of enga",
          "text_after_change": "a scientist, Fortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of enga"
        },
        "position": {
          "start_index": 9667,
          "end_index": 9681
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Cloud hosting\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n",
          "following_text": "Building a sustainable cloud for AI workloads\n“The level of engagement from th"
        },
        "change": {
          "type": "insert",
          "new_text": "[Firmus logo]\n"
        },
        "verification": {
          "text_before_change": "ortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\nBuilding a sustainable cloud for AI workloads\n“The level of engagement from th",
          "text_after_change": "ortune 500 financial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of engagement from th"
        },
        "position": {
          "start_index": 9682,
          "end_index": 9696
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "[Firmus logo]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ancial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\n",
          "following_text": "“The level of engagement from the Canonical team was remarkable. Even before w"
        },
        "change": {
          "type": "insert",
          "new_text": "Building a sustainable cloud for AI workloads\n"
        },
        "verification": {
          "text_before_change": "ancial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\n“The level of engagement from the Canonical team was remarkable. Even before w",
          "text_after_change": "ancial services company\n\n[Get the full case study]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of engagement from the Canonical team was remarkable. Even before w"
        },
        "position": {
          "start_index": 9696,
          "end_index": 9742
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Building a sustainable cloud for AI workloads\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "udy]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n",
          "following_text": "\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nM"
        },
        "change": {
          "type": "insert",
          "new_text": "“The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around OEMs and hardware choices. They were dedicated to our project from the get-go.”\n"
        },
        "verification": {
          "text_before_change": "udy]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nM",
          "text_after_change": "udy]\n\nCloud hosting\n[Firmus logo]\nBuilding a sustainable cloud for AI workloads\n“The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around OEMs and hardware choices. They were dedicated to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nM"
        },
        "position": {
          "start_index": 9742,
          "end_index": 9980
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "“The level of engagement from the Canonical team was remarkable. Even before we entered into a commercial agreement, Canonical offered valuable advice around OEMs and hardware choices. They were dedicated to our project from the get-go.”\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "Ms and hardware choices. They were dedicated to our project from the get-go.”\n",
          "following_text": "—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMe"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "Ms and hardware choices. They were dedicated to our project from the get-go.”\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMe",
          "text_after_change": "Ms and hardware choices. They were dedicated to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMe"
        },
        "position": {
          "start_index": 9980,
          "end_index": 9981
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "s and hardware choices. They were dedicated to our project from the get-go.”\n\n",
          "following_text": "[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives"
        },
        "change": {
          "type": "insert",
          "new_text": "—Tim Rosenfield, CEO and Co-Founder, Firmus\n"
        },
        "verification": {
          "text_before_change": "s and hardware choices. They were dedicated to our project from the get-go.”\n\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives",
          "text_after_change": "s and hardware choices. They were dedicated to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives"
        },
        "position": {
          "start_index": 9981,
          "end_index": 10025
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "—Tim Rosenfield, CEO and Co-Founder, Firmus\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": " our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n",
          "following_text": "Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives "
        },
        "change": {
          "type": "insert",
          "new_text": "["
        },
        "verification": {
          "text_before_change": " our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\nDownload the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives ",
          "text_after_change": " our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives "
        },
        "position": {
          "start_index": 10025,
          "end_index": 10026
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "["
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[",
          "following_text": "]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Pa"
        },
        "change": {
          "type": "insert",
          "new_text": "Download the full case study"
        },
        "verification": {
          "text_before_change": "our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Pa",
          "text_after_change": "our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Pa"
        },
        "position": {
          "start_index": 10026,
          "end_index": 10054
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Download the full case study"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study",
          "following_text": "\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Part"
        },
        "change": {
          "type": "insert",
          "new_text": "]\n"
        },
        "verification": {
          "text_before_change": "”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Part",
          "text_after_change": "”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Part"
        },
        "position": {
          "start_index": 10054,
          "end_index": 10056
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "\ufffd\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n",
          "following_text": "Media and\u0026 entertainment\nMachine learning drives down operational costs\n“Partn"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "\ufffd\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partn",
          "text_after_change": "\ufffd\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partn"
        },
        "position": {
          "start_index": 10057,
          "end_index": 10058
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\n",
          "following_text": "and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering "
        },
        "change": {
          "type": "insert",
          "new_text": "Media "
        },
        "verification": {
          "text_before_change": "\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nand\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering ",
          "text_after_change": "\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering "
        },
        "position": {
          "start_index": 10060,
          "end_index": 10066
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Media "
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "im Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia ",
          "following_text": "\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering wit"
        },
        "change": {
          "type": "insert",
          "new_text": "and"
        },
        "verification": {
          "text_before_change": "im Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia \u0026 entertainment\nMachine learning drives down operational costs\n“Partnering wit",
          "text_after_change": "im Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering wit"
        },
        "position": {
          "start_index": 10066,
          "end_index": 10069
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "and"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.8ydhlats9cc1",
        "anchor": {
          "preceding_text": "to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia ",
          "following_text": " entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets us concentrate on our co"
        },
        "change": {
          "type": "replace",
          "original_text": "\u0026",
          "new_text": "and"
        },
        "verification": {
          "text_before_change": "to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia \u0026 entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets us concentrate on our co",
          "text_after_change": "to our project from the get-go.”\n\n—Tim Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets us concentrate on our co"
        },
        "position": {
          "start_index": 10066,
          "end_index": 10070
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "and"
          },
          {
            "type": "delete",
            "original_text": "\u0026"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and",
          "following_text": " entertainment\nMachine learning drives down operational costs\n“Partnering with"
        },
        "change": {
          "type": "insert",
          "new_text": "\u0026"
        },
        "verification": {
          "text_before_change": "Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and entertainment\nMachine learning drives down operational costs\n“Partnering with",
          "text_after_change": "Rosenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering with"
        },
        "position": {
          "start_index": 10069,
          "end_index": 10070
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\u0026"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "osenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026",
          "following_text": "Machine learning drives down operational costs\n“Partnering with Canonical lets"
        },
        "change": {
          "type": "insert",
          "new_text": " entertainment\n"
        },
        "verification": {
          "text_before_change": "osenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026Machine learning drives down operational costs\n“Partnering with Canonical lets",
          "text_after_change": "osenfield, CEO and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets"
        },
        "position": {
          "start_index": 10070,
          "end_index": 10085
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " entertainment\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\n",
          "following_text": "“Partnering with Canonical lets us concentrate on our core business. Our data "
        },
        "change": {
          "type": "insert",
          "new_text": "Machine learning drives down operational costs\n"
        },
        "verification": {
          "text_before_change": "and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\n“Partnering with Canonical lets us concentrate on our core business. Our data ",
          "text_after_change": "and Co-Founder, Firmus\n[Download the full case study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets us concentrate on our core business. Our data "
        },
        "position": {
          "start_index": 10086,
          "end_index": 10133
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Machine learning drives down operational costs\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n",
          "following_text": "\n—Machine learning engineer, entertainment technology company\n\n[Download the f"
        },
        "change": {
          "type": "insert",
          "new_text": "“Partnering with Canonical lets us concentrate on our core business. Our data scientists can focus on data manipulation and model training rather than managing infrastructure.”\n"
        },
        "verification": {
          "text_before_change": "study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n\n—Machine learning engineer, entertainment technology company\n\n[Download the f",
          "text_after_change": "study]\n\nMedia and\u0026 entertainment\nMachine learning drives down operational costs\n“Partnering with Canonical lets us concentrate on our core business. Our data scientists can focus on data manipulation and model training rather than managing infrastructure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the f"
        },
        "position": {
          "start_index": 10133,
          "end_index": 10310
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "“Partnering with Canonical lets us concentrate on our core business. Our data scientists can focus on data manipulation and model training rather than managing infrastructure.”\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "on data manipulation and model training rather than managing infrastructure.”\n",
          "following_text": "—Machine learning engineer, entertainment technology company\n\n[Download the fu"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "on data manipulation and model training rather than managing infrastructure.”\n—Machine learning engineer, entertainment technology company\n\n[Download the fu",
          "text_after_change": "on data manipulation and model training rather than managing infrastructure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the fu"
        },
        "position": {
          "start_index": 10310,
          "end_index": 10311
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "n data manipulation and model training rather than managing infrastructure.”\n\n",
          "following_text": "\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downlo"
        },
        "change": {
          "type": "insert",
          "new_text": "—Machine learning engineer, entertainment technology company\n"
        },
        "verification": {
          "text_before_change": "n data manipulation and model training rather than managing infrastructure.”\n\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downlo",
          "text_after_change": "n data manipulation and model training rather than managing infrastructure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downlo"
        },
        "position": {
          "start_index": 10311,
          "end_index": 10372
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "—Machine learning engineer, entertainment technology company\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "rastructure.”\n\n—Machine learning engineer, entertainment technology company\n",
          "following_text": "[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downloa"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "rastructure.”\n\n—Machine learning engineer, entertainment technology company\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downloa",
          "text_after_change": "rastructure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, downloa"
        },
        "position": {
          "start_index": 10372,
          "end_index": 10373
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "astructure.”\n\n—Machine learning engineer, entertainment technology company\n\n",
          "following_text": "Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download"
        },
        "change": {
          "type": "insert",
          "new_text": "["
        },
        "verification": {
          "text_before_change": "astructure.”\n\n—Machine learning engineer, entertainment technology company\n\nDownload the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download",
          "text_after_change": "astructure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download"
        },
        "position": {
          "start_index": 10373,
          "end_index": 10374
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "["
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "structure.”\n\n—Machine learning engineer, entertainment technology company\n\n[",
          "following_text": "]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLAT"
        },
        "change": {
          "type": "insert",
          "new_text": "Download the full case study"
        },
        "verification": {
          "text_before_change": "structure.”\n\n—Machine learning engineer, entertainment technology company\n\n[]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLAT",
          "text_after_change": "structure.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLAT"
        },
        "position": {
          "start_index": 10374,
          "end_index": 10402
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Download the full case study"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "arning engineer, entertainment technology company\n\n[Download the full case study",
          "following_text": "\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE "
        },
        "change": {
          "type": "insert",
          "new_text": "]\n"
        },
        "verification": {
          "text_before_change": "arning engineer, entertainment technology company\n\n[Download the full case study\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE ",
          "text_after_change": "arning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE "
        },
        "position": {
          "start_index": 10402,
          "end_index": 10404
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "]\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ning engineer, entertainment technology company\n\n[Download the full case study]\n",
          "following_text": "\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE B"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ning engineer, entertainment technology company\n\n[Download the full case study]\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE B",
          "text_after_change": "ning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE B"
        },
        "position": {
          "start_index": 10405,
          "end_index": 10406
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ing engineer, entertainment technology company\n\n[Download the full case study]\n\n",
          "following_text": "TEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BA"
        },
        "change": {
          "type": "insert",
          "new_text": "\n"
        },
        "verification": {
          "text_before_change": "ing engineer, entertainment technology company\n\n[Download the full case study]\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BA",
          "text_after_change": "ing engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BA"
        },
        "position": {
          "start_index": 10407,
          "end_index": 10408
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 18,
        "table_id": "table-18",
        "table_title": "TEMPLATE CTA",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.auuyj7skq0ns",
        "anchor": {
          "preceding_text": "ture.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\n",
          "following_text": "TEMPLATE BASIC TEXT\nCost-effective, security-maintained, and supportedSecurity across all layers of the stack\n\nCanonical"
        },
        "change": {
          "type": "delete",
          "original_text": "Jumpstart your AI journey, download the MLOps toolkit\n\n\n"
        },
        "verification": {
          "text_before_change": "ture.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nCost-effective, security-maintained, and supportedSecurity across all layers of the stack\n\nCanonical",
          "text_after_change": "ture.”\n\n—Machine learning engineer, entertainment technology company\n\n[Download the full case study]\n\n\nTEMPLATE CTA\nTEMPLATE BASIC TEXT\nCost-effective, security-maintained, and supportedSecurity across all layers of the stack\n\nCanonical"
        },
        "position": {
          "start_index": 10424,
          "end_index": 10481
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Jumpstart your AI journey, download the MLOps toolkit"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 4
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 19,
        "table_id": "table-19",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 1,
        "column_index": 2,
        "column_header": "Canonical maintains and supports all the open sour...",
        "row_header": "Cost-effective, security-maintained, and supported..."
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.130xiu76y8ea",
        "anchor": {
          "preceding_text": "ownload the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "\n\nCanonical maintains and supports all the open source tooling in your AI infrastructure stack, including Kubernetes and"
        },
        "change": {
          "type": "replace",
          "original_text": "Security across all layers of the stack",
          "new_text": "Cost-effective, security-maintained, and supported"
        },
        "verification": {
          "text_before_change": "ownload the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nSecurity across all layers of the stack\n\nCanonical maintains and supports all the open source tooling in your AI infrastructure stack, including Kubernetes and",
          "text_after_change": "ownload the full case study]\n\n\nTEMPLATE CTA\nJumpstart your AI journey, download the MLOps toolkit\n\n\nTEMPLATE BASIC TEXT\nCost-effective, security-maintained, and supported\n\nCanonical maintains and supports all the open source tooling in your AI infrastructure stack, including Kubernetes and"
        },
        "position": {
          "start_index": 10504,
          "end_index": 10593
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Cost-effective, security-maintained, and supported"
          },
          {
            "type": "delete",
            "original_text": "Security across all layers of the stack"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.cs6ugj8q1nwv",
        "anchor": {
          "preceding_text": "ooling in your AI infrastructure stack, including Kubernetes and open source cloud management solutions like OpenStack. ",
          "following_text": "Fast-track compliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on aver"
        },
        "change": {
          "type": "insert",
          "new_text": "\n\nEnterprise support for the entire stack is available with a transparent subscription, on a transparent, per node basis via the Ubuntu Pro subscription, so you have full control over your TCO.\n\n"
        },
        "verification": {
          "text_before_change": "ooling in your AI infrastructure stack, including Kubernetes and open source cloud management solutions like OpenStack. Fast-track compliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on aver",
          "text_after_change": "ooling in your AI infrastructure stack, including Kubernetes and open source cloud management solutions like OpenStack. \n\nEnterprise support for the entire stack is available with a transparent subscription, on a transparent, per node basis via the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on aver"
        },
        "position": {
          "start_index": 10770,
          "end_index": 10965
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "Enterprise support for the entire stack is available "
          },
          {
            "type": "insert",
            "new_text": "with a transparent subscription, "
          },
          {
            "type": "insert",
            "new_text": "on a transparent, per node basis via the"
          },
          {
            "type": "insert",
            "new_text": " Ubuntu Pro"
          },
          {
            "type": "insert",
            "new_text": " subscription"
          },
          {
            "type": "insert",
            "new_text": ", so you have full control over your TCO.\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 9
      },
      {
        "id": "suggest.grfmnflrinw0",
        "anchor": {
          "preceding_text": "netes and open source cloud management solutions like OpenStack. \n\nEnterprise support for the entire stack is available ",
          "following_text": " Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely. L"
        },
        "change": {
          "type": "replace",
          "original_text": "on a transparent, per node basis via the",
          "new_text": "with a transparent subscription, "
        },
        "verification": {
          "text_before_change": "netes and open source cloud management solutions like OpenStack. \n\nEnterprise support for the entire stack is available on a transparent, per node basis via the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely. L",
          "text_after_change": "netes and open source cloud management solutions like OpenStack. \n\nEnterprise support for the entire stack is available with a transparent subscription,  Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely. L"
        },
        "position": {
          "start_index": 10825,
          "end_index": 10898
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "with a transparent subscription, "
          },
          {
            "type": "delete",
            "original_text": "on a transparent, per node basis via the"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.ldfdupynwc1s",
        "anchor": {
          "preceding_text": " a transparent subscription, on a transparent, per node basis via the Ubuntu Pro",
          "following_text": ", so you have full control over your TCO.\n\nFast-track compliance and run AI proj"
        },
        "change": {
          "type": "delete",
          "original_text": " subscription"
        },
        "verification": {
          "text_before_change": " a transparent subscription, on a transparent, per node basis via the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI proj",
          "text_after_change": " a transparent subscription, on a transparent, per node basis via the Ubuntu Pro, so you have full control over your TCO.\n\nFast-track compliance and run AI proj"
        },
        "position": {
          "start_index": 10909,
          "end_index": 10922
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": " subscription"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.vyixyxb90od6",
        "anchor": {
          "preceding_text": "the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely",
          "following_text": ".\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnbl"
        },
        "change": {
          "type": "replace",
          "original_text": " with critical CVEs fixed in under 24h on average",
          "new_text": ". Let us handle the CVEs"
        },
        "verification": {
          "text_before_change": "the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely with critical CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnbl",
          "text_after_change": "the Ubuntu Pro subscription, so you have full control over your TCO.\n\nFast-track compliance and run AI projects securely. Let us handle the CVEs.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnbl"
        },
        "position": {
          "start_index": 11015,
          "end_index": 11088
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": ". Let us handle the CVEs"
          },
          {
            "type": "delete",
            "original_text": " with critical CVEs fixed in under 24h on average"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.bqemmr3l9tkg",
        "anchor": {
          "preceding_text": "mpliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on average.\n\nExplore",
          "following_text": " \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeelin"
        },
        "change": {
          "type": "replace",
          "original_text": "  open source security with Canonical",
          "new_text": " Ubuntu Pro "
        },
        "verification": {
          "text_before_change": "mpliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on average.\n\nExplore  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeelin",
          "text_after_change": "mpliance and run AI projects securely. Let us handle the CVEs with critical CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeelin"
        },
        "position": {
          "start_index": 11098,
          "end_index": 11146
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " Ubuntu Pro"
          },
          {
            "type": "delete",
            "original_text": " "
          },
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "delete",
            "original_text": " "
          },
          {
            "type": "delete",
            "original_text": "open source security with Canonical"
          }
        ],
        "atomic_count": 5
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 20,
        "table_id": "table-20",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 1,
        "column_index": 2,
        "column_header": "Feeling stuck? Our consulting services and MLOps w...",
        "row_header": "Start your journey with a workshopUnblock your AI/..."
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.20dbhfhlfl6w",
        "anchor": {
          "preceding_text": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardware level. Building o"
        },
        "change": {
          "type": "insert",
          "new_text": "Start your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will Leveraging our in-house expertise, we work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n"
        },
        "verification": {
          "text_before_change": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardware level. Building o",
          "text_after_change": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will Leveraging our in-house expertise, we work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardware level. Building o"
        },
        "position": {
          "start_index": 11174,
          "end_index": 11678
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Start your journey with a workshop"
          },
          {
            "type": "insert",
            "new_text": "Unblock your AI"
          },
          {
            "type": "insert",
            "new_text": "/ML"
          },
          {
            "type": "insert",
            "new_text": " initiatives with"
          },
          {
            "type": "insert",
            "new_text": " an"
          },
          {
            "type": "insert",
            "new_text": " MLOps workshop"
          },
          {
            "type": "insert",
            "new_text": " \n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "Feeling stuck? Our "
          },
          {
            "type": "insert",
            "new_text": "consulting services and "
          },
          {
            "type": "insert",
            "new_text": "MLOps "
          },
          {
            "type": "insert",
            "new_text": "workshop can help you move forward.\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "Our experts will "
          },
          {
            "type": "insert",
            "new_text": "Leveraging our in-house expertise, we "
          },
          {
            "type": "insert",
            "new_text": "work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment"
          },
          {
            "type": "insert",
            "new_text": " – "
          },
          {
            "type": "insert",
            "new_text": "—"
          },
          {
            "type": "insert",
            "new_text": "private"
          },
          {
            "type": "insert",
            "new_text": " cloud"
          },
          {
            "type": "insert",
            "new_text": ", public"
          },
          {
            "type": "insert",
            "new_text": " cloud"
          },
          {
            "type": "insert",
            "new_text": ","
          },
          {
            "type": "insert",
            "new_text": " or"
          },
          {
            "type": "insert",
            "new_text": " hybrid"
          },
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "insert",
            "new_text": ", or multi-"
          },
          {
            "type": "insert",
            "new_text": "cloud"
          },
          {
            "type": "insert",
            "new_text": "."
          },
          {
            "type": "insert",
            "new_text": ".\n"
          },
          {
            "type": "insert",
            "new_text": "\n"
          },
          {
            "type": "insert",
            "new_text": "["
          },
          {
            "type": "insert",
            "new_text": "Learn more about our workshop"
          },
          {
            "type": "insert",
            "new_text": "]"
          },
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "insert",
            "new_text": "Contact the team"
          },
          {
            "type": "insert",
            "new_text": " \u003e"
          },
          {
            "type": "insert",
            "new_text": "\n"
          }
        ],
        "atomic_count": 39
      },
      {
        "id": "suggest.43vnzwlrvkxa",
        "anchor": {
          "preceding_text": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\n",
          "following_text": " \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will Leveraging our"
        },
        "change": {
          "type": "replace",
          "original_text": "Unblock your AI/ML initiatives with an MLOps workshop",
          "new_text": "Start your journey with a workshop"
        },
        "verification": {
          "text_before_change": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will Leveraging our",
          "text_after_change": "cal CVEs fixed in under 24h on average.\n\nExplore Ubuntu Pro  open source security with Canonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will Leveraging our"
        },
        "position": {
          "start_index": 11174,
          "end_index": 11261
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Start your journey with a workshop"
          },
          {
            "type": "delete",
            "original_text": "Unblock your AI"
          },
          {
            "type": "delete",
            "original_text": "/ML"
          },
          {
            "type": "delete",
            "original_text": " initiatives with"
          },
          {
            "type": "delete",
            "original_text": " an"
          },
          {
            "type": "delete",
            "original_text": " MLOps workshop"
          }
        ],
        "atomic_count": 6
      },
      {
        "id": "suggest.a1kh5gc9exd5",
        "anchor": {
          "preceding_text": "nonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI",
          "following_text": " initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and"
        },
        "change": {
          "type": "delete",
          "original_text": "/ML"
        },
        "verification": {
          "text_before_change": "nonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and",
          "text_after_change": "nonical \u003e\n\nTEMPLATE BASIC TEXT\nStart your journey with a workshopUnblock your AI initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and"
        },
        "position": {
          "start_index": 11223,
          "end_index": 11226
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "/ML"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.7t7kshsqg6ni",
        "anchor": {
          "preceding_text": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with",
          "following_text": " MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can "
        },
        "change": {
          "type": "insert",
          "new_text": " an"
        },
        "verification": {
          "text_before_change": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can ",
          "text_after_change": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can "
        },
        "position": {
          "start_index": 11243,
          "end_index": 11246
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " an"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.ob7dqfl5bbej",
        "anchor": {
          "preceding_text": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our ",
          "following_text": "workshop can help you move forward.\n\nOur experts will Leveraging our in-house expertise, we work with you to define an o"
        },
        "change": {
          "type": "replace",
          "original_text": "MLOps ",
          "new_text": "consulting services and "
        },
        "verification": {
          "text_before_change": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our MLOps workshop can help you move forward.\n\nOur experts will Leveraging our in-house expertise, we work with you to define an o",
          "text_after_change": "BASIC TEXT\nStart your journey with a workshopUnblock your AI/ML initiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and workshop can help you move forward.\n\nOur experts will Leveraging our in-house expertise, we work with you to define an o"
        },
        "position": {
          "start_index": 11284,
          "end_index": 11314
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "consulting services and "
          },
          {
            "type": "delete",
            "original_text": "MLOps "
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.gpoqc3u3lzs9",
        "anchor": {
          "preceding_text": "tiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\n",
          "following_text": "work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient "
        },
        "change": {
          "type": "replace",
          "original_text": "Leveraging our in-house expertise, we ",
          "new_text": "Our experts will "
        },
        "verification": {
          "text_before_change": "tiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nLeveraging our in-house expertise, we work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient ",
          "text_after_change": "tiatives with an MLOps workshop \n\nFeeling stuck? Our consulting services and MLOps workshop can help you move forward.\n\nOur experts will work with you to define an optimal architecture tailored to your existing ecosystem and implement a scalable, resilient "
        },
        "position": {
          "start_index": 11351,
          "end_index": 11406
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Our experts will "
          },
          {
            "type": "delete",
            "original_text": "Leveraging our in-house expertise, we "
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.t7288r4l3ngd",
        "anchor": {
          "preceding_text": "chitecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment",
          "following_text": "private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE"
        },
        "change": {
          "type": "replace",
          "original_text": "—",
          "new_text": " – "
        },
        "verification": {
          "text_before_change": "chitecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment—private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE",
          "text_after_change": "chitecture tailored to your existing ecosystem and implement a scalable, resilient infrastructure across any environment – private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE"
        },
        "position": {
          "start_index": 11563,
          "end_index": 11567
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " – "
          },
          {
            "type": "delete",
            "original_text": "—"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.sxxzartw67p7",
        "anchor": {
          "preceding_text": "ement a scalable, resilient infrastructure across any environment – —private",
          "following_text": ", public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  C"
        },
        "change": {
          "type": "delete",
          "original_text": " cloud"
        },
        "verification": {
          "text_before_change": "ement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  C",
          "text_after_change": "ement a scalable, resilient infrastructure across any environment – —private, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  C"
        },
        "position": {
          "start_index": 11574,
          "end_index": 11580
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": " cloud"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.nl88j392k498",
        "anchor": {
          "preceding_text": "le, resilient infrastructure across any environment – —private cloud, public",
          "following_text": ", or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the tea"
        },
        "change": {
          "type": "delete",
          "original_text": " cloud"
        },
        "verification": {
          "text_before_change": "le, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the tea",
          "text_after_change": "le, resilient infrastructure across any environment – —private cloud, public, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the tea"
        },
        "position": {
          "start_index": 11588,
          "end_index": 11594
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": " cloud"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.c5hmitzq1cr",
        "anchor": {
          "preceding_text": "ilient infrastructure across any environment – —private cloud, public cloud,",
          "following_text": " hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n"
        },
        "change": {
          "type": "insert",
          "new_text": " or"
        },
        "verification": {
          "text_before_change": "ilient infrastructure across any environment – —private cloud, public cloud, hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n",
          "text_after_change": "ilient infrastructure across any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n"
        },
        "position": {
          "start_index": 11595,
          "end_index": 11598
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " or"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.l6rnnth4u5nh",
        "anchor": {
          "preceding_text": " and implement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid",
          "following_text": "cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on "
        },
        "change": {
          "type": "replace",
          "original_text": ", or multi-",
          "new_text": " "
        },
        "verification": {
          "text_before_change": " and implement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid, or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on ",
          "text_after_change": " and implement a scalable, resilient infrastructure across any environment – —private cloud, public cloud, or hybrid cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on "
        },
        "position": {
          "start_index": 11605,
          "end_index": 11617
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " "
          },
          {
            "type": "delete",
            "original_text": ", or multi-"
          }
        ],
        "atomic_count": 2
      },
      {
        "id": "suggest.38q1ak9usm63",
        "anchor": {
          "preceding_text": "s any environment – —private cloud, public cloud, or hybrid , or multi-cloud",
          "following_text": ".\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConf"
        },
        "change": {
          "type": "delete",
          "original_text": "."
        },
        "verification": {
          "text_before_change": "s any environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConf",
          "text_after_change": "s any environment – —private cloud, public cloud, or hybrid , or multi-cloud.\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConf"
        },
        "position": {
          "start_index": 11622,
          "end_index": 11623
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "."
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.hgwhxc98zm7w",
        "anchor": {
          "preceding_text": " environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[",
          "following_text": "]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on"
        },
        "change": {
          "type": "insert",
          "new_text": "Learn more about our workshop"
        },
        "verification": {
          "text_before_change": " environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on",
          "text_after_change": " environment – —private cloud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on"
        },
        "position": {
          "start_index": 11627,
          "end_index": 11656
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": "Learn more about our workshop"
          }
        ],
        "atomic_count": 1
      },
      {
        "id": "suggest.muru29czqov8",
        "anchor": {
          "preceding_text": "oud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]",
          "following_text": " Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on U"
        },
        "change": {
          "type": "insert",
          "new_text": " "
        },
        "verification": {
          "text_before_change": "oud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop] Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on U",
          "text_after_change": "oud, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on U"
        },
        "position": {
          "start_index": 11657,
          "end_index": 11658
        },
        "atomic_changes": [
          {
            "type": "insert",
            "new_text": " "
          }
        ],
        "atomic_count": 1
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 21,
        "table_id": "table-21",
        "table_title": "TEMPLATE BASIC TEXT",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.51l7m8dlpm1d",
        "anchor": {
          "preceding_text": "d, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\n",
          "following_text": "\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUn"
        },
        "change": {
          "type": "delete",
          "original_text": "Confidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardware level. Building on Ubuntu confidential VMs, you can now safeguard your sensitive data and intellectual property with a hardware-rooted execution environment that spans both the CPU and GPU.\n\nUbuntu confidential VMs are available on Azure, Google Cloud and AWS.\n\nYou can enable confidential computing in the datacenter and at the edge thanks to Ubuntu Intel TDX build, which comes with all the required pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n"
        },
        "verification": {
          "text_before_change": "d, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\nConfidential AI \n\n\nConfidential AI on Ubuntu protects data in use at the hardware level. Building on Ubuntu confidential VMs, you can now safeguard your sensitive data and intellectual property with a hardware-rooted execution environment that spans both the CPU and GPU.\n\nUbuntu confidential VMs are available on Azure, Google Cloud and AWS.\n\nYou can enable confidential computing in the datacenter and at the edge thanks to Ubuntu Intel TDX build, which comes with all the required pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUn",
          "text_after_change": "d, public cloud, or hybrid , or multi-cloud..\n\n[Learn more about our workshop]  Contact the team \u003e\n\nTEMPLATE BASIC TEXT\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUn"
        },
        "position": {
          "start_index": 11703,
          "end_index": 12262
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Confidential AI \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Confidential AI on Ubuntu protects data in use at the hardware level. Building on Ubuntu confidential VMs, you can now safeguard your sensitive data and intellectual property with a hardware-rooted execution environment that spans both the CPU and GPU.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Ubuntu confidential VMs are available on Azure, Google Cloud and AWS.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "You can enable confidential computing in the datacenter and at the edge thanks to Ubuntu Intel TDX build, which comes with all the required pieces for both the guest and host.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Read more about confidential AI"
          },
          {
            "type": "delete",
            "original_text": " \u003e"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 13
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 22,
        "table_id": "table-22",
        "table_title": "TEMPLATE CTA",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": "which comes with all the required pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\n",
          "following_text": "\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-time space trac"
        },
        "change": {
          "type": "delete",
          "original_text": "Build your cloud with Canonical \n\n\n"
        },
        "verification": {
          "text_before_change": "which comes with all the required pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-time space trac",
          "text_after_change": "which comes with all the required pieces for both the guest and host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-time space trac"
        },
        "position": {
          "start_index": 12279,
          "end_index": 12315
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Build your cloud with Canonical"
          },
          {
            "type": "delete",
            "original_text": " \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 4
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 23,
        "table_id": "table-23",
        "table_title": "TEMPLATE LEARN MORE FOOTER",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": " host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\n",
          "following_text": "TEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here "
        },
        "change": {
          "type": "delete",
          "original_text": "Open source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-time space tracking with AI/ML supercomputing\nLearn how University of Tasmania is modernising its space-tracking data processing with the Firmus Supercloud, built on Canonical’s open AI infrastructure stack.\n\nMachine learning drives down operational costs in media and entertainment industry\nDiscover how a global entertainment technology leader is putting Canonical Managed Kubeflow at the heart of a modernised AI strategy.\n\nAI on Private Cloud: why is it relevant in the hyperscalers era?\nWatch the webinar to learn how a private cloud can address critical AI challenges including cost optimisation, digital sovereignty and performance.\n\nRun AI at scale with NVIDIA DGX and Charmed Kubeflow\nRunning AI at scale requires seamlessly integrated application and hardware layers. Read the whitepaper to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\n"
        },
        "verification": {
          "text_before_change": " host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nOpen source AI infrastructure in action\n\nUniversity of Tasmania unlocks real-time space tracking with AI/ML supercomputing\nLearn how University of Tasmania is modernising its space-tracking data processing with the Firmus Supercloud, built on Canonical’s open AI infrastructure stack.\n\nMachine learning drives down operational costs in media and entertainment industry\nDiscover how a global entertainment technology leader is putting Canonical Managed Kubeflow at the heart of a modernised AI strategy.\n\nAI on Private Cloud: why is it relevant in the hyperscalers era?\nWatch the webinar to learn how a private cloud can address critical AI challenges including cost optimisation, digital sovereignty and performance.\n\nRun AI at scale with NVIDIA DGX and Charmed Kubeflow\nRunning AI at scale requires seamlessly integrated application and hardware layers. Read the whitepaper to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here ",
          "text_after_change": " host.\n\nRead more about confidential AI \u003e\n\n\nTEMPLATE CTA\nBuild your cloud with Canonical \n\n\n\nTEMPLATE LEARN MORE FOOTER\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here "
        },
        "position": {
          "start_index": 12346,
          "end_index": 13330
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Open source AI infrastructure in action\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "University of Tasmania unlocks real-time space tracking with AI/ML supercomputing"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Learn how University of Tasmania is modernising its space-tracking data processing with the Firmus Supercloud, built on Canonical’s open AI infrastructure stack.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Machine learning drives down operational costs in media and entertainment industry"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Discover how a global entertainment technology leader is putting Canonical Managed Kubeflow at the heart of a modernised AI strategy.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "AI on Private Cloud: why is it relevant in the hyperscalers era?"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Watch the webinar to learn how a private cloud can address critical AI challenges including cost optimisation, digital sovereignty and performance.\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Run AI at scale with NVIDIA DGX and Charmed Kubeflow"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "Running AI at scale requires seamlessly integrated application and hardware layers. Read the whitepaper to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 20
      }
    ]
  },
  {
    "location": {
      "section": "Body",
      "parent_heading": "What customers say",
      "heading_level": 1,
      "in_table": true,
      "table": {
        "table_index": 24,
        "table_id": "table-24",
        "table_title": "TEMPLATE CTA",
        "row_index": 0,
        "column_index": 0,
        "column_header": "",
        "row_header": ""
      },
      "in_metadata": false
    },
    "suggestions": [
      {
        "id": "suggest.4xhb2u5stty3",
        "anchor": {
          "preceding_text": "to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nTEMPLATE CTA\n",
          "following_text": "\n\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack for AI infrastructure at scale\n\nWe make "
        },
        "change": {
          "type": "delete",
          "original_text": "Send us a message to discuss your AI infrastructure needs \n\n\n"
        },
        "verification": {
          "text_before_change": "to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nTEMPLATE CTA\nSend us a message to discuss your AI infrastructure needs \n\n\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack for AI infrastructure at scale\n\nWe make ",
          "text_after_change": "to see how NVIDIA DGX and Charmed Kubeflow can provide an optimised foundation for your AI operations. \n\n\n\nTEMPLATE CTA\n\n\n\n\n\nOLD design\n\n\n—Visible page starts here —\nTEMPLATE HERO\nYour full stack for AI infrastructure at scale\n\nWe make "
        },
        "position": {
          "start_index": 13346,
          "end_index": 13408
        },
        "atomic_changes": [
          {
            "type": "delete",
            "original_text": "Send us a message to discuss your AI infrastructure needs"
          },
          {
            "type": "delete",
            "original_text": " \n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          },
          {
            "type": "delete",
            "original_text": "\n"
          }
        ],
        "atomic_count": 4
      }
    ]
  }
]
```
