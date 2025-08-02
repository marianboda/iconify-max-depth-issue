# Iconify Max Depth Issue Reproduction

> 🤖 **Developed with [Claude Code](https://claude.ai/code)** | **Related**: [Iconify Issue #386](https://github.com/iconify/iconify/issues/386)

Minimal SvelteKit reproduction for `effect_update_depth_exceeded` error in @iconify/svelte.

## 🎯 Key Findings

| Configuration         | Incremental Loading | Direct Navigation |
| --------------------- | ------------------- | ----------------- |
| **Without PaneForge** | ✅ 2000+ icons      | ❌ 1500+ icons    |
| **With PaneForge**    | ❌ 1400 icons       | ❌ 1400 icons     |

> ⚠️ **Note**: Crash thresholds may vary on different machines/setups depending on performance and browser configuration.

**Root Causes**:

1. **PaneForge interaction** (primary) - breaks both loading methods at 1400 icons
2. **Iconify bulk initialization** (secondary) - overwhelms Svelte 5's effect system on direct navigation

## 🧪 Quick Test

```bash
npm install && npm run dev
```

**Test URLs**:

- `?count=1400&paneforge=false` ✅ Works
- `?count=1500&paneforge=false` ❌ Crashes (direct nav)
- `?count=1400&paneforge=true` ❌ Crashes (PaneForge)

**Interactive Testing**: Use checkbox and input field to compare incremental vs direct loading

## 🔬 Technical Details

**Error**: `effect_update_depth_exceeded` in `@iconify/svelte:1763`
**Stack**: SvelteKit + Svelte 5 + @iconify/svelte + PaneForge
**Issue**: Infinite update loop in icon loading during bulk initialization

## 💡 Workarounds

1. **Avoid PaneForge** with many icons (primary fix)
2. **Use incremental loading** vs direct navigation
3. **Stay under 1400 icons** if PaneForge required
4. **Implement lazy loading** for large icon sets

## 🚀 Commands

```bash
npm run dev    # Start dev server
npm run check  # Type checking
npm run build  # Production build
```

## 📁 Structure

```
src/routes/+page.svelte  # Main reproduction test
CLAUDE.md               # Technical docs
```
