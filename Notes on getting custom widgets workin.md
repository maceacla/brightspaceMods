# Notes on getting custom widgets working on brightspace

Collecting workspace information

Based on the workspace, here's how the Brightspace widgets and GitHub integration works:

### Brightspace Widget Configuration

1. **Widget Structure**
- Brightspace uses a custom widget framework with `d2l-widget` components
- Two main widgets are configured:
  - diary-widget-wrapper.html
  - title-card-wrapper.html



2. **Widget Loading**
```html
<div class="d2l-widget-iframe-wrapper">
    <div class="d2l-iframe-loading-container"></div>
    <iframe class="d2l-iframe d2l-iframe-fit" 
            src="diary-widget.html"
            frameborder="0"
            allowfullscreen>
    </iframe>
</div>
```

### GitHub Integration Technique

1. **Content Storage**
- Widget code stored in GitHub repository at:
  ```
  /elec4707w2025/widgetCode/
    ├── diary-widget.html
    └── title-card-widget.html
  ```
- Content stored separately:
  ```
  /elec4707w2025/content/
    └── diary.md
  ```

2. **Loading Mechanism**
```javascript
const widgetUrl = 'https://raw.githubusercontent.com/maceacla/brightspaceMods/refs/heads/main/elec4707w2025/widgetCode/diary-widget.html';
const response = await fetch(widgetUrl);
const widgetHTML = await response.text();
```

3. **Key Benefits**
- Version control for widget code
- Separation of content and presentation
- Easy updates without Brightspace admin access
- CDN-like distribution via GitHub's raw content URLs

This approach allows maintaining widget code and content externally while integrating seamlessly with Brightspace's widget framework.


# Diary Widget: Analysis of Widget Changes

### Previous Issues
1. Script loading order
2. Dependency management
3. Asynchronous loading conflicts

### Key Changes Made

1. **Dependency Management**
   - Moved `marked.js` loading to wrapper
   - Ensures library exists before render attempt

2. **Render Flow**
   ```mermaid
   graph TD
      A[Load marked.js] --> B[Wait for marked]
      B --> C[Load widget HTML]
      C --> D[Fetch markdown]
      D --> E[Render content]
   ```

3. **Content Loading**
   - Wrapper now handles full content lifecycle
   - Eliminates race conditions
   - Ensures dependencies exist

### Why It Works
- Single responsibility for each component:
  - Wrapper: Handles dependencies and orchestration
  - Widget HTML: Provides structure and styling
  - Markdown file: Contains content only
- Proper async/await flow prevents timing issues
- No iframe means direct DOM access
- Dependencies guaranteed before render attempt

### Technical Implementation
```javascript


<div id="widget-container"></div>
<script src="marked.js"></script>  // Load dependency first
<script>
    (async () => {
        await waitForMarked();     // Ensure dependency loaded
        await loadWidgetHTML();    // Load structure
        await fetchAndRenderMD();  // Handle content
    })();
</script>
```


# Title-Card Widget: Different Approach

Here's why the title-card widget works in an iframe despite being loaded differently:

### Current Implementation

The title-card widget works through a two-layer approach:

1. **Iframe Level**


title-card-wrapper.html

 is loaded in Brightspace's iframe with this structure:
```html
<div class="d2l-widget-iframe-wrapper">
    <div class="d2l-iframe-loading-container"></div>
    <iframe class="d2l-iframe d2l-iframe-fit" 
            src="title-card-wrapper.html"
            frameborder="0"
            allowfullscreen>
    </iframe>
</div>
```

2. **Widget Content**
The title-card-widget.html is simpler than the diary widget because:
- No external dependencies (like marked.js)
- Static HTML/CSS content
- No dynamic content loading

### Why It Works
1. **Containment**: The `widget-root` class uses `all: initial` to create a clean styling context
2. **Self-contained**: The widget doesn't rely on external libraries or dynamic content
3. **Scoped Styling**: CSS is scoped within the widget root div
4. **Simple Loading**: Single fetch operation to get static HTML content

Unlike the diary widget that needs markdown processing, the title-card widget is self-contained and doesn't require complex dependency management or content processing.