# Iconify Max Depth Issue Reproduction

A minimal SvelteKit reproduction case for the `effect_update_depth_exceeded` error in @iconify/svelte when using many icons with PaneForge components.

**Related Issue**: [Iconify GitHub Issue #386](https://github.com/iconify/iconify/issues/386)

## 🐛 Issue Summary

The @iconify/svelte library throws a `effect_update_depth_exceeded` error when rendering many icons under specific conditions. This repository provides a minimal reproduction case that identifies **two distinct crash patterns**.

## 🎯 Key Findings

### Pattern 1: Multiple Panes Issue
- **❌ Crashes**: 500+ icons per pane when distributed across multiple PaneForge panes
- **Threshold**: 500 icons × 3 panes = 1500 total icons triggers crash
- **Root Cause**: Interaction between PaneForge pane management and Iconify's state management

### Pattern 2: Initial Bulk Loading Issue  
- **❌ Crashes**: 1000+ icons in single pane when loaded on initial page load
- **✅ Works**: Same icon count when reached incrementally through user interaction
- **Root Cause**: Iconify's initialization phase cannot handle simultaneous processing of many icons

## 🧪 How to Reproduce

### Prerequisites

```bash
npm install
npm run dev
```

### Test Cases

#### Test Case 1: Multiple Panes (Distributed Icons)
1. Switch to 3-pane layout (modify code to restore 3 panes)
2. Set icon count to 500 per pane
3. **Result**: `effect_update_depth_exceeded` error

#### Test Case 2: Single Pane Initial Load
1. Edit `src/routes/+page.svelte` and set `iconCount = $state(1500)`
2. Refresh page  
3. **Result**: `effect_update_depth_exceeded` error on page load

#### Test Case 3: Single Pane Incremental
1. Start with `iconCount = $state(100)`
2. Use the input field to gradually increase to 3000
3. **Result**: No errors, works perfectly

## 🔧 Current Configuration

The repository is currently set up to test **Pattern 2** (single pane configuration):
- **Pane 1**: Empty (for comparison)  
- **Pane 2**: Contains all test icons
- **Default**: 100 icons (safe starting point)
- **Interactive**: Adjustable via number input

## 📊 Detailed Thresholds

| Configuration | Icon Count | Method | Result |
|---------------|------------|--------|---------|
| 3 Panes | 100 per pane (300 total) | Any | ✅ Works |
| 3 Panes | 500 per pane (1500 total) | Any | ❌ Crashes |
| 1 Pane | 1000+ icons | Initial load | ❌ Crashes |
| 1 Pane | 3000+ icons | Incremental | ✅ Works |

## 🔬 Technical Details

### Error Message
```
effect_update_depth_exceeded
Maximum update depth exceeded. This typically indicates that an effect reads and writes the same piece of state
```

### Error Location
- **File**: `@iconify/svelte` 
- **Line**: 1763 (in `$effect` block)
- **Suggests**: Infinite update loop in icon loading/rendering logic

### Environment
- **Framework**: SvelteKit with Svelte 5
- **Icons**: @iconify/svelte (on-demand loading from Iconify API)
- **Panes**: PaneForge for resizable layouts
- **Syntax**: Modern Svelte 5 runes (`$state`, `$derived`)

## 💡 Implications for Developers

### Workarounds
1. **Avoid multiple panes** with many icons each
2. **Consolidate icons** into fewer panes when possible  
3. **Load incrementally** rather than bulk initial loading
4. **Implement lazy loading** with IntersectionObserver

### For Iconify Maintainers
This reproduction case provides:
- **Isolated test environment** for debugging
- **Two distinct scenarios** to investigate
- **Clear thresholds** for triggering the issue
- **Evidence** that it's related to initialization timing, not just icon count

## 🚀 Development Commands

```bash
# Start development server
npm run dev

# Run type checking  
npm run check

# Format code
npm run format

# Lint code
npm run lint

# Build for production
npm run build
```

## 📝 Project Structure

```
├── src/
│   ├── routes/
│   │   ├── +layout.svelte     # Root layout with favicon
│   │   └── +page.svelte       # Main reproduction test page
│   └── lib/
│       └── assets/            # Static assets
├── CLAUDE.md                  # Detailed technical documentation
└── README.md                  # This file
```

## 🤝 Contributing

This repository serves as a minimal reproduction case. To help debug:

1. **Test additional scenarios** and document results
2. **Profile performance** during crashes
3. **Experiment with different icon sets** or loading patterns
4. **Report findings** to [Iconify Issue #386](https://github.com/iconify/iconify/issues/386)
