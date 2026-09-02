# Portfolio Management Guide

This guide explains how your Jekyll portfolio is structured and provides step-by-step instructions for adding, moving, editing, and managing projects and categories.

---

## 1. Architecture Overview

Your portfolio is organized into clean, modular Jekyll collections:

| Directory / File | Description |
| :--- | :--- |
| `_projects/` | Contains all **Active Projects** displayed on the Home page (`/`) and Projects tab (`/projects/`). |
| `_archive/` | Contains all **Archived Projects** displayed on the Archive tab (`/archive/`). |
| `_includes/project-card.html` | Shared reusable card component used by both Projects and Archive sections. |
| `_includes/projects.html` | Renders the grid for `site.projects`. |
| `_includes/archive.html` | Renders the grid for `site.archive`. |
| `projects.html` | Page template routing to `/projects/`. |
| `archive.html` | Page template routing to `/archive/`. |
| `_config.yml` | Configures collections, navigation data, and theme settings. |

---

## 2. How to Move Projects (Active $\leftrightarrow$ Archive)

Because each project is self-contained in its own folder, moving a project between Active and Archive takes just a single command or folder move.

### Moving a Project to Archive
To move a project from Active to Archive (e.g., `my-project`):
```bash
git mv _projects/my-project _archive/my-project
```
*(Or simply drag and drop the folder in your file explorer / IDE from `_projects/` into `_archive/`)*.

### Restoring an Archived Project to Active
To move a project back to Active:
```bash
git mv _archive/my-project _projects/my-project
```

Jekyll automatically updates the lists, links, and image references immediately upon restart or reload.

---

## 3. How to Add a New Project

### Step 1: Create the Project Directory
Create a new folder inside either `_projects/` (for an active project) or `_archive/` (for an archived project). Use lowercase and hyphens:
```
_projects/my-new-project/
```

### Step 2: Add Images and Assets
Place your project's cover image and gallery images directly inside the new project folder:
```
_projects/my-new-project/
├── index.md
├── cover.jpg
└── detail-photo.jpg
```

### Step 3: Create `index.md`
Create `index.md` inside your project folder using this frontmatter template:

```markdown
{% raw %}
---
layout: post
title: My New Project Title
description: A 1-3 sentence summary of the project that appears on the card preview and the header of the detail page.
skills: 
  - Python
  - CAD
  - Embedded Systems
  - 3D Printing

main-image: /cover.jpg
---

---
## Project Overview
Write your project background and details here in Markdown.

---
## Photo Gallery:
{% include image-gallery.html images="detail-photo.jpg" height="400" %}

---
## Embedded YouTube Video (Optional):
{% include youtube-video.html id="VIDEO_ID_HERE" autoplay="false" %}

---
## Interactive 3D CAD Model (Optional):
{% include model-viewer.html model="/_projects/my-project/model.glb" alt="3D CAD Model" height="480" %}

---
## Technical Code or Analysis Snippet:
```python
def example():
    print("Embedded code block")
```
{% endraw %}
```

---

## 4. How to Customize Project Cards

The cards rendered on both the Home, Projects, and Archive pages share a single include file:
[`_includes/project-card.html`](_includes/project-card.html).

If you want to customize the look, add a date badge, or change the button text across all cards, edit `_includes/project-card.html` once. The changes automatically reflect across all sections without duplicating code.

---

## 5. How to Add a New Navigation Tab / Category

To add another category (such as `Research`, `Publications`, or activating `Volunteering`):

1. **Register the collection in `_config.yml`**:
   ```yaml
   collections:
     projects:
       output: true
       permalink: /projects/:path/
     archive:
       output: true
       permalink: /archive/:path/
     research:
       output: true
       permalink: /research/:path/

   include:
     - _projects
     - _archive
     - _research
   ```

2. **Create the category folder**:
   Create `_research/` and add project folders inside it.

3. **Create the include file `_includes/research.html`**:
   ```html
   {% raw %}
   <section id="research" class="project-grid-section">
       <h2 class="section-title">Research</h2>
       <div class="grid-container">
           {% for item in site.research %}
               {% include project-card.html item=item %}
           {% endfor %}
       </div>
   </section>
   {% endraw %}
   ```

4. **Create the page file `research.html` in the root**:
   ```html
   {% raw %}
   ---
   layout: wrapper
   permalink: /research/
   title: Research
   ---

   {% include research.html %}
   {% endraw %}
   ```

5. **Add the link to the navbar (`_includes/navbar.html`) and footer (`_includes/footer.html`)**:
   ```html
   <a href="/research/">
       <span>Research</span>
   </a>
   ```

---

## 6. Running the Local Preview Server

You can run the live preview server at any time to see changes in real time.

In PowerShell:
```powershell
$env:Path = "C:\msys64\ucrt64\bin;C:\msys64\usr\bin;C:\Ruby33-x64\bin;" + $env:Path
bundle exec jekyll serve --livereload --port 4000
```

- Open your browser to: `http://localhost:4000`
- The server supports **LiveReload**: when you edit any file, the browser automatically refreshes to display your updates.
- Press `Ctrl+C` in the terminal to stop the server.

---

## 7. Embedding Interactive 3D CAD Models

The website features a dedicated Three.js WebGL CAD rendering engine with real-time **Shaded with Edges** outline rendering, multi-material preservation, studio lighting, smooth continuous auto-rotation, and microscopic near-clipping for inspection.

### Step 1: Convert CAD (`.step`) to Multi-Material `.glb`
You can convert any STEP CAD assembly while preserving all distinct face materials and body colors using Python (`trimesh` + `cascadio`):

```powershell
python -c "
import trimesh, numpy as np
scene = trimesh.load('path/to/model.step')
new_scene = trimesh.Scene()
for node_name in scene.graph.nodes_geometry:
    transform, geom_name = scene.graph[node_name]
    geom = scene.geometry[geom_name]
    if hasattr(geom.visual, 'material') and hasattr(geom.visual.material, 'materials') and len(geom.visual.material.materials) > 1 and hasattr(geom.visual, 'face_materials'):
        fm = np.array(geom.visual.face_materials)
        for u in np.unique(fm):
            sub = geom.submesh([np.where(fm == u)[0]], append=True)
            sub.visual.material = geom.visual.material.materials[u]
            new_scene.add_geometry(sub, node_name=f'{node_name}_sub{u}', transform=transform)
    else:
        new_scene.add_geometry(geom, node_name=node_name, transform=transform)
open('path/to/model.glb', 'wb').write(new_scene.export(file_type='glb'))
"
```

### Step 2: Embed the Model
In your project's `index.md`:
```markdown
{% raw %}
## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/my-project/model.glb" alt="My Board 3D CAD" height="500" pitch="-90" %}
{% endraw %}
```

#### Optional Attributes:
- `height`: Height in pixels (default: `"500"`).
- `pitch`: Initial rotation about the X-axis in degrees (e.g. `pitch="-90"`).
- `yaw`: Initial rotation about the Y-axis in degrees (e.g. `yaw="45"`).
- `roll`: Initial rotation about the Z-axis in degrees.
- `alt`: Title displayed in the viewer badge.

---

## 8. Deploying Updates to GitHub Pages

Whenever you are ready to publish your changes:
```bash
git add .
git commit -m "Update portfolio projects and navigation"
git push origin dev
```
If your live site tracks `main`, merge `dev` into `main` and push to deploy to `https://AlexZidar.github.io`.
