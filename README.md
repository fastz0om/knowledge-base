# Knowledge Base - Obsidian

Personal knowledge base for documenting problems, solutions, and learnings.

## Structure

```
.
├── Attachments/          # Files and images
├── Daily Notes/          # Daily notes (auto-generated)
├── Notes/                # Main notes folder
│   ├── Index.md         # Starting point
│   ├── Problems.md      # Problems index
│   ├── Solutions.md     # Solutions index
│   └── Knowledge Base.md # General knowledge
└── Templates/            # Note templates
    ├── Daily Note Template.md
    ├── Problem Template.md
    ├── Solution Template.md
    └── Knowledge Note Template.md
```

## How to Run

### Setup

1. Open Obsidian
2. Click "Open folder as vault"
3. Select this directory (`/home/fastz0om/projects/obsidian`)
4. Settings are already configured in `.obsidian/`

### Configuration

Settings are pre-configured:
- **Attachments**: Saved to `Attachments/` folder
- **New files**: Created in `Notes/` folder
- **Daily notes**: Template and folder configured
- **Templates**: Folder configured

### Usage

#### Daily Notes
- `Ctrl+P` → "Daily Note" - Create today's note
- Automatically uses template from `Templates/Daily Note Template.md`

#### Creating Notes

**Problem Note:**
1. `Ctrl+I` → Select "Problem Template"
2. Fill in problem details
3. Tag with `#problem` to appear in Problems index

**Solution Note:**
1. `Ctrl+I` → Select "Solution Template"
2. Link to related problem using `[[]]`
3. Tag with `#solution`

**Knowledge Note:**
1. `Ctrl+I` → Select "Knowledge Note Template"
2. Organize by category

### Keyboard Shortcuts

- `Ctrl+P` - Command palette
- `Ctrl+I` - Insert template
- `Ctrl+O` - Open link
- `Ctrl+G` - Open graph view
- `Ctrl+E` - Toggle edit/preview

### Best Practices

1. **Tagging**: Use consistent tags (`#problem`, `#solution`, `#knowledge`)
2. **Linking**: Use `[[]]` to create connections between notes
3. **Templates**: Always use templates for consistency
4. **Daily Notes**: Review and organize daily notes weekly
5. **Attachments**: Keep files organized in `Attachments/` folder

### Recommended Plugins

Consider installing these community plugins:
- **Tag Wrangler** - Better tag management
- **Dataview** - Query and display notes
- **Calendar** - Calendar view for daily notes
- **Templater** - Advanced templating (alternative to built-in)

## CI Stages

Not applicable - this is a local knowledge base.

## Notes

- All configuration files are in `.obsidian/`
- Templates use date/time variables (supported by Templater plugin)
- Use markdown links `[[]]` for note linking
- Daily notes format: `YYYY-MM-DD`

