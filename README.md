# Iconify Max Depth Issue Reproduction

A minimal SvelteKit reproduction case for the `effect_update_depth_exceeded` error in @iconify/svelte when using many icons with PaneForge components.

> **Note**: This reproduction case was developed with significant assistance from 🤖 [Claude Code](https://claude.ai/code) for analysis, testing, and documentation.

**Related Issue**: [Iconify GitHub Issue #386](https://github.com/iconify/iconify/issues/386)

## 🐛 Issue Summary

The @iconify/svelte library throws a `effect_update_depth_exceeded` error when rendering many icons under specific conditions. This repository provides a minimal reproduction case that identifies **three distinct crash patterns**.

## 🎯 Key Findings

### Pattern 1: PaneForge Interaction Issue ⚠️ **PRIMARY CAUSE**

- **❌ With PaneForge**: Crashes at ~1400 icons regardless of loading method
- **✅ Without PaneForge**: Up to 2000+ icons work with incremental loading
- **Root Cause**: **PaneForge interaction breaks both incremental and direct loading**

### Pattern 2: Pure Iconify Initialization Issue

- **❌ Direct Navigation**: Crashes at 1500+ icons without PaneForge on initial page load
- **✅ Incremental Loading**: Works fine up to 2000+ icons without PaneForge
- **Root Cause**: Iconify's bulk initialization overwhelms Svelte 5's effect system

### Pattern 3: Loading Method Makes the Difference

- **Incremental Loading**: Allows Iconify to process icons gradually as user increases count
- **Direct Navigation**: Forces simultaneous processing during page initialization
- **Key Insight**: The issue is about initialization timing, not just total icon count

## 🧪 How to Reproduce

### Prerequisites

```bash
npm install
npm run dev
```

### Test Cases

#### Test Case 1: PaneForge Impact (Most Important!)

1. **Without PaneForge + Incremental**: Start at 100, increase to 2000+ icons ✅ Works
2. **Without PaneForge + Direct**: Navigate to `?count=1500&paneforge=false` ❌ Crashes
3. **With PaneForge + Incremental**: Start at 100, increase to 1400 icons ❌ Crashes  
4. **With PaneForge + Direct**: Navigate to `?count=1400&paneforge=true` ❌ Crashes
5. **Conclusion**: PaneForge breaks both loading methods at 1400 icons

#### Test Case 2: Loading Method Comparison (Without PaneForge)

1. **Incremental**: Use input field to go 100 → 500 → 1000 → 2000 ✅ Works
2. **Direct Navigation**: Go directly to `?count=1500&paneforge=false` ❌ Crashes
3. **Conclusion**: Direct navigation fails at 1500+, incremental works at 2000+

#### Test Case 3: URL Parameter Testing

Use these URLs to test different scenarios:
- `?count=1400&paneforge=false` - ✅ Works (incremental safe, direct safe)
- `?count=1500&paneforge=false` - ❌ Crashes (direct navigation)
- `?count=1400&paneforge=true` - ❌ Crashes (PaneForge breaks everything)
- `?count=2000&paneforge=false` - ❌ Crashes (direct navigation high volume)

## 🔧 Current Configuration

The repository includes a **toggle to test with/without PaneForge**:

- **Checkbox**: "Use PaneForge" - unchecked by default for comparison
- **Without PaneForge**: Simple grid layout for baseline testing
- **With PaneForge**: Dual-pane layout (empty pane + icons pane)
- **Default**: 100 icons (safe starting point)
- **Interactive**: Adjustable via number input

## 📊 Detailed Test Results

| Configuration | Loading Method | Icon Count | Result | Notes |
|---------------|----------------|------------|--------|--------|
| **No PaneForge** | Incremental | 2000+ | ✅ Works | Safe method |
| **No PaneForge** | Direct Navigation | 1400 | ✅ Works | Safe threshold |
| **No PaneForge** | Direct Navigation | 1500+ | ❌ Crashes | Initialization overload |
| **With PaneForge** | Incremental | 1400 | ❌ Crashes | PaneForge breaks everything |
| **With PaneForge** | Direct Navigation | 1400 | ❌ Crashes | PaneForge breaks everything |

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

1. **Avoid PaneForge** when using many icons (primary fix)
2. **Use incremental loading** instead of direct navigation to high icon counts
3. **Stay under 1400 icons** if PaneForge is required
4. **Implement lazy loading** with IntersectionObserver for large icon sets

### For Iconify Maintainers

This reproduction case provides:

- **Isolated test environment** for debugging both PaneForge and pure Iconify issues
- **Clear evidence** that PaneForge interaction is the primary trigger
- **Proof** that initialization timing matters more than total icon count
- **Systematic test cases** with specific thresholds and loading methods

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
