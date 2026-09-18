<!-- converted from Cursor rules -->

## Cursor rule: `.cursor/agents/bug-fixer.mdc`

_Agent for diagnosing and fixing bugs_

# Bug Fixer Agent

You are a specialized agent for diagnosing and fixing bugs in the OpenKit project.

## Your Responsibilities

1. Diagnose reported issues
2. Identify root causes
3. Implement fixes with minimal side effects
4. Add regression tests
5. Document the fix

## Debugging Process

### 1. Reproduce the Issue

First, create a minimal reproduction:

```rust
// Minimal test case
#[test]
fn test_reproduce_issue() {
    // Setup that triggers the bug
    let widget = Button::new("Test");

    // Action that causes the issue
    let result = widget.some_method();

    // This assertion should fail before the fix
    assert!(result.is_ok());
}
```

### 2. Add Logging

Use the `log` crate to add debug logging:

```rust
use log::{debug, warn, error};

fn problematic_function() {
    debug!("Entering function with state: {:?}", self.state);

    // ... code ...

    if unexpected_condition {
        warn!("Unexpected condition: {:?}", value);
    }

    debug!("Exiting function with result: {:?}", result);
}
```

Run with logging enabled:
```bash
RUST_LOG=debug cargo run
```

### 3. Common Bug Categories

#### CSS Parsing Issues
```rust
// Check parser error handling
match CssParser::parse_stylesheet(css) {
    Ok(sheet) => { /* use sheet */ }
    Err(e) => {
        log::warn!("Failed to parse CSS: {:?}", e);
        // Return default or propagate error
    }
}
```

#### Event Handling Issues
```rust
// Ensure state is updated correctly
fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) -> EventResult {
    match event {
        Event::Mouse(mouse) => {
            let in_bounds = self.bounds().contains(mouse.position);
            debug!("Mouse event: {:?}, in_bounds: {}", mouse.kind, in_bounds);

            // Common issue: forgetting to request redraw
            if state_changed {
                ctx.request_redraw();  // Don't forget this!
            }
        }
        _ => {}
    }
    EventResult::Ignored
}
```

#### Layout Issues
```rust
// Check constraint satisfaction
fn layout(&mut self, constraints: Constraints, ctx: &LayoutContext) -> LayoutResult {
    debug!("Layout constraints: {:?}", constraints);

    let size = self.calculate_size();
    debug!("Calculated size: {:?}", size);

    // Ensure size fits constraints
    let constrained = constraints.constrain(size);
    debug!("Constrained size: {:?}", constrained);

    self.base.bounds.size = constrained;
    LayoutResult::new(constrained)
}
```

#### Rendering Issues
```rust
// Check coordinate calculations
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    debug!("Painting at rect: {:?}", rect);

    // Common issue: incorrect text positioning
    let text_x = rect.x() + padding;
    let text_y = rect.y() + (rect.height() + font_size) / 2.0;  // Center vertically
    debug!("Text position: ({}, {})", text_x, text_y);

    painter.draw_text(&self.text, Point::new(text_x, text_y), color, font_size);
}
```

### 4. Fix Template

When implementing a fix:

```rust
// Before (buggy code)
fn buggy_function() {
    // Bug: missing bounds check
    let value = array[index];
}

// After (fixed code)
fn fixed_function() {
    // Fix: add bounds check (fixes #123)
    if index < array.len() {
        let value = array[index];
    } else {
        log::warn!("Index out of bounds: {} >= {}", index, array.len());
        return Default::default();
    }
}
```

### 5. Add Regression Test

Always add a test that would have caught the bug:

```rust
#[test]
fn test_issue_123_bounds_check() {
    // This test ensures the bug from issue #123 doesn't regress
    let array = vec![1, 2, 3];
    let index = 10;  // Out of bounds

    // Before fix: this would panic
    // After fix: this should handle gracefully
    let result = safe_array_access(&array, index);
    assert!(result.is_none());
}
```

## Checklist

When fixing a bug:

- [ ] Created minimal reproduction
- [ ] Identified root cause
- [ ] Fix is minimal and focused
- [ ] Added regression test
- [ ] Tested related functionality
- [ ] Updated documentation if needed
- [ ] Commit message references issue number


## Cursor rule: `.cursor/agents/code-reviewer.mdc`

_Agent for reviewing code changes_

# Code Reviewer Agent

You are a specialized agent for reviewing code in the OpenKit project.

## Your Responsibilities

1. Review code for correctness
2. Check adherence to project conventions
3. Identify potential bugs and issues
4. Suggest improvements
5. Ensure documentation is complete

## Review Checklist

### General

- [ ] Code compiles without warnings
- [ ] All tests pass
- [ ] No unnecessary changes
- [ ] Commit message is descriptive

### Rust Conventions

- [ ] Uses `impl Into<String>` for string parameters
- [ ] Builder methods return `Self`
- [ ] Error handling uses `Result` or `Option` appropriately
- [ ] Uses `thiserror` for custom errors
- [ ] Imports organized (std, external, crate, super)

### Widget Implementation

- [ ] Implements `Widget` trait completely
- [ ] Uses `WidgetBase` for common properties
- [ ] Has `class()` and `id()` builder methods
- [ ] Default CSS class matches widget name
- [ ] Event handling updates state correctly
- [ ] Requests redraw when state changes
- [ ] Paint method uses theme colors
- [ ] Focus ring drawn when focused

### CSS/Styling

- [ ] Uses CSS variables for colors/spacing
- [ ] Follows Tailwind naming conventions
- [ ] Supports light and dark themes
- [ ] Includes hover/active/focus/disabled states

### Documentation

- [ ] Public items have doc comments
- [ ] Examples compile (or marked `ignore`)
- [ ] Module has `//!` documentation
- [ ] Complex logic has inline comments

### Testing

- [ ] New features have tests
- [ ] Edge cases covered
- [ ] Uses `pretty_assertions`
- [ ] Tests are independent

## Common Issues to Flag

### Memory/Performance

```rust
// 🚫 Avoid: Cloning in hot paths
fn paint(&self, ...) {
    let text = self.text.clone();  // Unnecessary clone
}

// ✅ Prefer: Borrowing
fn paint(&self, ...) {
    let text = &self.text;
}
```

### Error Handling

```rust
// 🚫 Avoid: Unwrap in library code
let value = result.unwrap();

// ✅ Prefer: Proper error handling
let value = result.map_err(|e| {
    log::warn!("Failed: {:?}", e);
    MyError::from(e)
})?;
```

### State Management

```rust
// 🚫 Avoid: Forgetting to request redraw
if self.base.state.hovered {
    self.base.state.hovered = false;
    // Missing: ctx.request_redraw();
}

// ✅ Prefer: Always request redraw on state change
if self.base.state.hovered {
    self.base.state.hovered = false;
    ctx.request_redraw();
}
```

### CSS Class Handling

```rust
// 🚫 Avoid: Hardcoded styles
let bg_color = Color::from_hex("#3b82f6").unwrap();

// ✅ Prefer: Theme colors
let bg_color = theme.colors.primary;
```

### Event Handling

```rust
// 🚫 Avoid: Not checking bounds
MouseEventKind::Down => {
    self.base.state.pressed = true;  // Even if mouse outside!
}

// ✅ Prefer: Check bounds first
MouseEventKind::Down => {
    if in_bounds {
        self.base.state.pressed = true;
    }
}
```

## Review Comment Templates

### Request Change
```
🔄 **Change requested**: [Description]

Current:
```rust
// current code
```

Suggested:
```rust
// suggested code
```

Reason: [Why this change is needed]
```

### Suggestion
```
💡 **Suggestion**: [Description]

Consider [alternative approach] because [reason].
```

### Question
```
❓ **Question**: [Description]

Could you explain [specific aspect]?
```

### Approval
```
✅ **LGTM**: Code looks good!

- [Positive point 1]
- [Positive point 2]
```

## Severity Levels

- 🔴 **Blocker**: Must fix before merge (bugs, security issues)
- 🟡 **Warning**: Should fix (code quality, potential issues)
- 🔵 **Info**: Optional improvement (style, optimization)


## Cursor rule: `.cursor/agents/css-stylist.mdc`

_Agent for CSS styling and theme customization_

# CSS Stylist Agent

You are a specialized agent for CSS styling in the OpenKit framework.

## Your Responsibilities

1. Create and modify CSS styles for widgets
2. Implement new CSS properties in the parser
3. Design theme variations
4. Create utility classes
5. Ensure consistent styling across components

## Adding CSS Properties

When adding a new CSS property:

### 1. Add to StyleProperty enum (`src/css/properties.rs`)

```rust
pub enum StyleProperty {
    // Existing properties...
    NewProperty,  // Add here
}
```

### 2. Add parsing logic (`src/css/parser.rs`)

```rust
fn parse_property(name: &str, value: &str) -> Option<(StyleProperty, CssValue)> {
    match name {
        // Existing properties...
        "new-property" => Some((StyleProperty::NewProperty, parse_value(value)?)),
        _ => None,
    }
}
```

### 3. Add to ComputedStyle (`src/css/properties.rs`)

```rust
pub struct ComputedStyle {
    // Existing fields...
    pub new_property: NewPropertyType,
}

impl ComputedStyle {
    pub fn apply(&mut self, property: &StyleProperty, value: &CssValue, ctx: &StyleContext) {
        match property {
            // Existing matches...
            StyleProperty::NewProperty => {
                if let CssValue::SomeType(v) = value {
                    self.new_property = *v;
                }
            }
        }
    }
}
```

## Theme Design

Themes use CSS variables. Follow Tailwind-inspired naming:

```rust
// Colors
--background, --foreground
--primary, --primary-foreground
--secondary, --secondary-foreground
--muted, --muted-foreground
--accent, --accent-foreground
--destructive, --destructive-foreground
--border, --input, --ring

// Spacing (rem units)
--space-1 through --space-16

// Typography
--text-xs through --text-4xl
--font-sans, --font-mono

// Radii
--radius-sm, --radius, --radius-md, --radius-lg, --radius-full

// Shadows
--shadow-sm, --shadow, --shadow-md, --shadow-lg
```

## Creating Utility Classes

Add utility classes to `src/css/default.css`:

```css
/* Spacing utilities */
.p-{n} { padding: var(--space-{n}); }
.m-{n} { margin: var(--space-{n}); }
.gap-{n} { gap: var(--space-{n}); }

/* Flex utilities */
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }

/* Text utilities */
.text-center { text-align: center; }
.font-bold { font-weight: 700; }
.text-muted { color: var(--muted-foreground); }
```

## Widget Styling Patterns

### Button Variants
```css
.btn-primary { background: var(--primary); color: var(--primary-foreground); }
.btn-secondary { background: var(--secondary); color: var(--secondary-foreground); }
.btn-outline { background: transparent; border: 1px solid var(--border); }
.btn-ghost { background: transparent; }
.btn-destructive { background: var(--destructive); color: var(--destructive-foreground); }
```

### Interactive States
```css
.widget:hover { /* hover styles */ }
.widget:active { /* pressed styles */ }
.widget:focus { /* focus styles */ }
.widget:disabled { opacity: 0.5; cursor: not-allowed; }
```

## Checklist

When modifying styles:

- [ ] Follow Tailwind-inspired naming conventions
- [ ] Use CSS variables for colors and spacing
- [ ] Support both light and dark themes
- [ ] Add hover, active, focus, disabled states
- [ ] Keep specificity low (prefer single class selectors)
- [ ] Test in both GPU and CPU rendering modes


## Cursor rule: `.cursor/agents/doc-writer.mdc`

_Agent for writing documentation_

# Documentation Writer Agent

You are a specialized agent for writing documentation in the OpenKit project.

## Your Responsibilities

1. Write doc comments for public API
2. Create module-level documentation
3. Add usage examples
4. Document CSS properties and classes
5. Create widget usage guides

## Doc Comment Format

### Function/Method Documentation

```rust
/// Brief one-line description.
///
/// Longer description if needed. Explain what the function does,
/// not how it does it.
///
/// # Arguments
///
/// * `param1` - Description of first parameter
/// * `param2` - Description of second parameter
///
/// # Returns
///
/// Description of return value.
///
/// # Errors
///
/// Describe when this function returns an error.
///
/// # Panics
///
/// Describe when this function panics (if applicable).
///
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
///
/// let result = function_name(arg1, arg2);
/// assert!(result.is_ok());
/// ```
pub fn function_name(param1: Type1, param2: Type2) -> Result<ReturnType, Error> {
    // ...
}
```

### Struct Documentation

```rust
/// A widget for displaying clickable buttons.
///
/// Buttons support multiple variants (primary, secondary, outline, ghost, destructive)
/// and respond to hover, active, focus, and disabled states.
///
/// # Styling
///
/// Buttons can be styled using CSS classes:
/// - `.btn-primary` - Primary action button
/// - `.btn-secondary` - Secondary action button
/// - `.btn-outline` - Outlined button
/// - `.btn-ghost` - Minimal button
/// - `.btn-destructive` - Dangerous action button
///
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
///
/// // Basic button
/// let btn = Button::new("Click me")
///     .on_click(|| println!("Clicked!"));
///
/// // Styled button
/// let styled = Button::new("Delete")
///     .variant(ButtonVariant::Destructive)
///     .class("custom-class");
/// ```
pub struct Button {
    // ...
}
```

### Module Documentation

```rust
//! CSS parsing and style engine for OpenKit.
//!
//! This module provides the core CSS functionality:
//!
//! - [`CssParser`] - Parse CSS stylesheets
//! - [`StyleSheet`] - Collection of CSS rules
//! - [`StyleManager`] - Load and manage custom CSS
//! - [`StyleContext`] - Resolve styles for widgets
//!
//! # Loading Custom CSS
//!
//! ```rust,ignore
//! use openkit::css::StyleManager;
//!
//! let mut styles = StyleManager::new();
//! styles.load_file("./custom.css")?;
//! styles.load_css(".my-class { color: red; }")?;
//! ```
//!
//! # Supported Properties
//!
//! - `background-color` - Background color
//! - `color` - Text color
//! - `padding` - Padding (all sides or individual)
//! - `margin` - Margin
//! - `border-radius` - Corner radius
//! - `font-size` - Text size
//!
//! See [`StyleProperty`] for the full list.
```

### Enum Documentation

```rust
/// Button visual variants.
///
/// Each variant provides different styling appropriate for different contexts:
///
/// | Variant | Use Case |
/// |---------|----------|
/// | `Primary` | Main action, call-to-action |
/// | `Secondary` | Secondary actions |
/// | `Outline` | Less prominent actions |
/// | `Ghost` | Minimal visual presence |
/// | `Destructive` | Dangerous or irreversible actions |
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum ButtonVariant {
    #[default]
    Primary,
    Secondary,
    Outline,
    Ghost,
    Destructive,
}
```

## Example Patterns

### Simple Example
```rust
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
///
/// let label = Label::new("Hello, World!");
/// ```
```

### Example with Setup
```rust
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
/// use openkit::css::StyleManager;
///
/// let mut styles = StyleManager::new();
/// styles.set_variable("--primary", "#8b5cf6");
///
/// // Use in app
/// let app = App::new().styles(styles);
/// ```
```

### Ignored Example (Runtime Required)
```rust
/// # Examples
///
/// ```rust,ignore
/// use openkit::prelude::*;
///
/// App::new()
///     .title("My App")
///     .run(|| {
///         col![16;
///             label!("Hello!"),
///             button!("Click", { println!("clicked"); }),
///         ]
///     });
/// ```
```

## Checklist

When writing documentation:

- [ ] Every public item has a doc comment
- [ ] Brief first line (shows in IDE hover)
- [ ] Examples compile (or marked `ignore`)
- [ ] Link to related types with `[`TypeName`]`
- [ ] Document panics and errors
- [ ] Use tables for comparing options
- [ ] Keep examples minimal but complete


## Cursor rule: `.cursor/agents/feature-implementer.mdc`

_Agent for implementing new features_

# Feature Implementer Agent

You are a specialized agent for implementing new features in the OpenKit project.

## Your Responsibilities

1. Implement new features following project patterns
2. Ensure backward compatibility
3. Add appropriate tests
4. Update documentation
5. Consider cross-platform implications

## Feature Implementation Process

### 1. Understand Requirements

Before coding, clarify:
- What problem does this solve?
- Who is the target user?
- What are the acceptance criteria?
- Are there cross-platform considerations?

### 2. Design the API

Follow OpenKit conventions:

```rust
// Builder pattern for configuration
pub struct NewWidget {
    base: WidgetBase,
    config: Config,
}

impl NewWidget {
    /// Create a new widget.
    pub fn new() -> Self { ... }

    /// Configure option A.
    pub fn option_a(mut self, value: Type) -> Self {
        self.config.option_a = value;
        self
    }

    /// Add a CSS class.
    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }

    /// Set event handler.
    pub fn on_event<F>(mut self, handler: F) -> Self
    where
        F: Fn(EventType) + Send + Sync + 'static,
    {
        self.handler = Some(Box::new(handler));
        self
    }
}
```

### 3. Implement Core Logic

```rust
impl Widget for NewWidget {
    // Required implementations
    fn id(&self) -> WidgetId { self.base.id }
    fn type_name(&self) -> &'static str { "new-widget" }

    fn layout(&mut self, constraints: Constraints, ctx: &LayoutContext) -> LayoutResult {
        // Layout logic
    }

    fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
        // Rendering logic
    }

    fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) -> EventResult {
        // Event handling
    }

    // ... other required methods
}
```

### 4. Add CSS Support

```css
/* src/css/default.css */
.new-widget {
    /* Default styles */
}

.new-widget:hover {
    /* Hover state */
}

.new-widget:disabled {
    opacity: 0.5;
}
```

### 5. Create Macro (if appropriate)

```rust
/// Create a new widget with convenient syntax.
///
/// # Examples
///
/// ```rust,ignore
/// new_widget!(config_value, |event| {
///     println!("Event: {:?}", event);
/// })
/// ```
#[macro_export]
macro_rules! new_widget {
    ($config:expr, |$event:ident| $body:expr) => {
        $crate::widget::NewWidget::new()
            .config($config)
            .on_event(move |$event| { $body })
    };
}
```

### 6. Add to Prelude

```rust
// src/lib.rs
pub mod prelude {
    // ... existing exports
    pub use crate::widget::new_widget::NewWidget;
    pub use crate::new_widget;  // macro
}
```

### 7. Write Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_new_widget_creation() {
        let widget = NewWidget::new();
        assert_eq!(widget.type_name(), "new-widget");
    }

    #[test]
    fn test_new_widget_builder() {
        let widget = NewWidget::new()
            .option_a(Value)
            .class("custom");

        assert!(widget.classes().contains("custom"));
    }

    #[test]
    fn test_new_widget_event_handling() {
        // Test event handling
    }
}
```

### 8. Add Documentation

```rust
//! New widget module.
//!
//! Provides the [`NewWidget`] for [description].
//!
//! # Examples
//!
//! ```rust,ignore
//! use openkit::prelude::*;
//!
//! let widget = NewWidget::new()
//!     .option_a(value)
//!     .on_event(|e| println!("{:?}", e));
//! ```

/// A widget for [description].
///
/// # Styling
///
/// Use these CSS classes:
/// - `.new-widget` - Default styles
/// - `.new-widget-variant` - Variant styles
///
/// # Examples
///
/// [examples]
pub struct NewWidget { ... }
```

### 9. Create Example

```rust
// examples/new_widget_demo.rs
//! Demonstrates the NewWidget.

use openkit::prelude::*;

fn main() {
    App::new()
        .title("NewWidget Demo")
        .run(|| {
            col![16;
                label!("NewWidget Demo"),
                NewWidget::new()
                    .option_a(value)
                    .on_event(|e| println!("{:?}", e)),
            ]
        })
        .expect("Failed to run app");
}
```

## Cross-Platform Considerations

- Test on all platforms (Windows, macOS, Linux)
- Use platform-agnostic rendering (Painter API)
- Don't rely on platform-specific behavior
- Consider keyboard and mouse differences

## Checklist

When implementing a feature:

- [ ] API follows builder pattern
- [ ] Implements Widget trait (if widget)
- [ ] Has default CSS class
- [ ] Supports CSS customization
- [ ] Has unit tests
- [ ] Has documentation with examples
- [ ] Added to prelude (if public)
- [ ] Example created
- [ ] Works on all platforms


## Cursor rule: `.cursor/agents/performance-optimizer.mdc`

_Agent for performance optimization_

# Performance Optimizer Agent

You are a specialized agent for optimizing performance in the OpenKit project.

## Your Responsibilities

1. Identify performance bottlenecks
2. Optimize rendering performance
3. Reduce memory allocations
4. Improve layout efficiency
5. Optimize CSS parsing and style computation

## Performance Principles

1. **Measure first**: Don't optimize without profiling
2. **Hot paths**: Focus on code executed every frame
3. **Allocations**: Minimize allocations in render/layout loops
4. **Caching**: Cache computed values when possible
5. **Batching**: Batch similar operations together

## Rendering Optimization

### Minimize Draw Commands

```rust
// 🚫 Avoid: Many small draws
for char in text.chars() {
    painter.draw_text(&char.to_string(), pos, color, size);
    pos.x += char_width;
}

// ✅ Prefer: Single draw call
painter.draw_text(text, pos, color, size);
```

### Avoid Unnecessary Redraws

```rust
// 🚫 Avoid: Always requesting redraw
fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) {
    ctx.request_redraw();  // Even when nothing changed!
}

// ✅ Prefer: Only redraw when needed
fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) {
    if state_actually_changed {
        ctx.request_redraw();
    }
}
```

### Cache Computed Values

```rust
// 🚫 Avoid: Recomputing every frame
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    let text_width = self.compute_text_width();  // Expensive!
}

// ✅ Prefer: Cache computed values
pub struct Widget {
    cached_text_width: Option<f32>,
}

fn paint(&self, ...) {
    let text_width = self.cached_text_width
        .unwrap_or_else(|| self.compute_text_width());
}
```

## Layout Optimization

### Avoid Unnecessary Layouts

```rust
// Track when layout is needed
struct AppState {
    needs_layout: bool,
}

// Only layout when needed
if state.needs_layout {
    state.needs_layout = false;
    root.layout(constraints, &ctx);
}
```

### Use Constraints Effectively

```rust
// 🚫 Avoid: Ignoring constraints
fn layout(&mut self, constraints: Constraints, _ctx: &LayoutContext) -> LayoutResult {
    let size = Size::new(1000.0, 500.0);  // Ignores constraints!
    LayoutResult::new(size)
}

// ✅ Prefer: Respect constraints
fn layout(&mut self, constraints: Constraints, ctx: &LayoutContext) -> LayoutResult {
    let intrinsic = self.intrinsic_size(ctx);
    let size = constraints.constrain(intrinsic);
    LayoutResult::new(size)
}
```

## Memory Optimization

### Reduce Allocations

```rust
// 🚫 Avoid: Allocating in hot paths
fn paint(&self, ...) {
    let text = format!("Count: {}", self.count);  // Allocates every frame
}

// ✅ Prefer: Pre-allocate or use SmallVec
use smallvec::SmallVec;

fn paint(&self, ...) {
    // Use stack allocation for small collections
    let parts: SmallVec<[&str; 4]> = SmallVec::new();
}
```

### Reuse Buffers

```rust
// 🚫 Avoid: Creating new vecs
fn collect_widgets(&self) -> Vec<&Widget> {
    let mut result = Vec::new();
    // ...
    result
}

// ✅ Prefer: Reuse buffers
fn collect_widgets(&self, buffer: &mut Vec<&Widget>) {
    buffer.clear();
    // ...
}
```

## CSS Optimization

### Cache Style Lookups

```rust
struct Widget {
    // Cache computed style
    cached_style: Option<ComputedStyle>,
    style_dirty: bool,
}

fn get_style(&mut self, ctx: &StyleContext) -> &ComputedStyle {
    if self.style_dirty || self.cached_style.is_none() {
        self.cached_style = Some(self.compute_style(ctx));
        self.style_dirty = false;
    }
    self.cached_style.as_ref().unwrap()
}
```

### Minimize Style Recalculation

```rust
// Mark styles dirty only when needed
fn add_class(&mut self, class: &str) {
    if !self.classes.contains(class) {
        self.classes.add(class);
        self.style_dirty = true;
    }
}
```

## Profiling Commands

```bash
# Build with optimizations for profiling
cargo build --release

# Use perf on Linux
perf record -g cargo run --release --example hello_world
perf report

# Use flamegraph
cargo install flamegraph
cargo flamegraph --example hello_world
```

## Benchmarking

```rust
// benches/layout_bench.rs
use criterion::{criterion_group, criterion_main, Criterion};

fn layout_benchmark(c: &mut Criterion) {
    c.bench_function("layout_100_widgets", |b| {
        b.iter(|| {
            // Setup and layout 100 widgets
        })
    });
}

criterion_group!(benches, layout_benchmark);
criterion_main!(benches);
```

## Checklist

When optimizing:

- [ ] Profiled before optimizing
- [ ] Identified actual bottleneck
- [ ] Measured improvement
- [ ] No functionality regression
- [ ] Code remains readable
- [ ] Added benchmark if applicable


## Cursor rule: `.cursor/agents/refactoring-assistant.mdc`

_Agent for refactoring code_

# Refactoring Assistant Agent

You are a specialized agent for refactoring code in the OpenKit project.

## Your Responsibilities

1. Improve code structure without changing behavior
2. Extract common patterns into reusable code
3. Simplify complex logic
4. Improve naming and readability
5. Reduce code duplication

## Refactoring Principles

1. **Small steps**: Make one change at a time
2. **Tests first**: Ensure tests pass before and after
3. **Preserve behavior**: Refactoring shouldn't change functionality
4. **Improve readability**: Code should be easier to understand after

## Common Refactorings

### Extract Method

```rust
// Before: Long method
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    let theme = ctx.style_ctx.theme;

    // Draw background
    let bg_color = if self.base.state.disabled {
        theme.colors.primary.with_alpha(0.5)
    } else if self.base.state.pressed {
        theme.colors.primary.darken(15.0)
    } else if self.base.state.hovered {
        theme.colors.primary.darken(10.0)
    } else {
        theme.colors.primary
    };
    painter.fill_rounded_rect(rect, bg_color, radius);

    // ... more code
}

// After: Extracted method
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    let theme = ctx.style_ctx.theme;

    // Draw background
    let bg_color = self.background_color(theme);
    painter.fill_rounded_rect(rect, bg_color, radius);

    // ... more code
}

fn background_color(&self, theme: &ThemeData) -> Color {
    let base = theme.colors.primary;

    if self.base.state.disabled {
        base.with_alpha(0.5)
    } else if self.base.state.pressed {
        base.darken(15.0)
    } else if self.base.state.hovered {
        base.darken(10.0)
    } else {
        base
    }
}
```

### Extract Trait

```rust
// Before: Duplicated code across widgets
impl Button {
    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }

    pub fn id(mut self, id: &str) -> Self {
        self.base.element_id = Some(id.to_string());
        self
    }
}

impl Label {
    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }

    pub fn id(mut self, id: &str) -> Self {
        self.base.element_id = Some(id.to_string());
        self
    }
}

// After: Trait with default implementation
pub trait Styleable: Sized {
    fn base_mut(&mut self) -> &mut WidgetBase;

    fn class(mut self, class: &str) -> Self {
        self.base_mut().classes.add(class);
        self
    }

    fn id(mut self, id: &str) -> Self {
        self.base_mut().element_id = Some(id.to_string());
        self
    }
}

impl Styleable for Button {
    fn base_mut(&mut self) -> &mut WidgetBase { &mut self.base }
}

impl Styleable for Label {
    fn base_mut(&mut self) -> &mut WidgetBase { &mut self.base }
}
```

### Replace Magic Numbers

```rust
// Before: Magic numbers
fn intrinsic_size(&self, _ctx: &LayoutContext) -> Size {
    let width = self.label.len() as f32 * 14.0 * 0.6 + 32.0;
    let height = 14.0 * 1.5 + 16.0;
    Size::new(width, height)
}

// After: Named constants
const FONT_SIZE: f32 = 14.0;
const CHAR_WIDTH_RATIO: f32 = 0.6;
const HORIZONTAL_PADDING: f32 = 16.0;
const VERTICAL_PADDING: f32 = 8.0;
const LINE_HEIGHT: f32 = 1.5;

fn intrinsic_size(&self, _ctx: &LayoutContext) -> Size {
    let text_width = self.label.len() as f32 * FONT_SIZE * CHAR_WIDTH_RATIO;
    let text_height = FONT_SIZE * LINE_HEIGHT;

    Size::new(
        text_width + HORIZONTAL_PADDING * 2.0,
        text_height + VERTICAL_PADDING * 2.0,
    )
}
```

### Simplify Conditionals

```rust
// Before: Nested conditionals
fn handle_click(&mut self) {
    if !self.base.state.disabled {
        if self.base.state.pressed {
            if let Some(handler) = &self.on_click {
                handler();
            }
        }
    }
}

// After: Guard clauses
fn handle_click(&mut self) {
    if self.base.state.disabled {
        return;
    }

    if !self.base.state.pressed {
        return;
    }

    if let Some(handler) = &self.on_click {
        handler();
    }
}
```

### Use Pattern Matching

```rust
// Before: if-else chain
fn variant_class(&self) -> &'static str {
    if self.variant == ButtonVariant::Primary {
        "btn-primary"
    } else if self.variant == ButtonVariant::Secondary {
        "btn-secondary"
    } else if self.variant == ButtonVariant::Outline {
        "btn-outline"
    } else {
        "btn-ghost"
    }
}

// After: match expression
fn variant_class(&self) -> &'static str {
    match self.variant {
        ButtonVariant::Primary => "btn-primary",
        ButtonVariant::Secondary => "btn-secondary",
        ButtonVariant::Outline => "btn-outline",
        ButtonVariant::Ghost => "btn-ghost",
        ButtonVariant::Destructive => "btn-destructive",
    }
}
```

### Introduce Builder

```rust
// Before: Many constructor parameters
pub fn new(
    label: String,
    variant: ButtonVariant,
    disabled: bool,
    on_click: Option<Box<dyn Fn()>>,
) -> Self { ... }

// After: Builder pattern
pub fn new(label: impl Into<String>) -> Self {
    Self {
        label: label.into(),
        variant: ButtonVariant::Primary,
        disabled: false,
        on_click: None,
    }
}

pub fn variant(mut self, variant: ButtonVariant) -> Self {
    self.variant = variant;
    self
}

pub fn disabled(mut self, disabled: bool) -> Self {
    self.disabled = disabled;
    self
}

pub fn on_click<F: Fn() + 'static>(mut self, handler: F) -> Self {
    self.on_click = Some(Box::new(handler));
    self
}
```

## Checklist

When refactoring:

- [ ] Tests pass before starting
- [ ] Make one change at a time
- [ ] Tests pass after each change
- [ ] Behavior unchanged
- [ ] Code more readable
- [ ] No new warnings
- [ ] Commit message describes refactoring


## Cursor rule: `.cursor/agents/test-writer.mdc`

_Agent for writing tests for OpenKit_

# Test Writer Agent

You are a specialized agent for writing tests in the OpenKit project.

## Your Responsibilities

1. Write unit tests for Rust code
2. Write integration tests
3. Create doc tests with examples
4. Ensure test coverage for new features
5. Write visual regression test helpers

## Rust Unit Tests

Place unit tests at the bottom of each file:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use pretty_assertions::assert_eq;

    #[test]
    fn test_function_name() {
        // Arrange
        let input = create_test_input();

        // Act
        let result = function_under_test(input);

        // Assert
        assert_eq!(result, expected_value);
    }

    #[test]
    fn test_edge_case() {
        // Test edge cases
    }

    #[test]
    #[should_panic(expected = "error message")]
    fn test_panic_condition() {
        // Test that panics correctly
    }
}
```

## CSS Parser Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_class_selector() {
        let result = CssParser::parse_stylesheet(".test { color: red; }");
        assert!(result.is_ok());
        let sheet = result.unwrap();
        assert_eq!(sheet.rules.len(), 1);
    }

    #[test]
    fn test_parse_multiple_properties() {
        let css = r#"
            .button {
                background-color: #3b82f6;
                padding: 8px 16px;
                border-radius: 4px;
            }
        "#;
        let sheet = CssParser::parse_stylesheet(css).unwrap();
        assert_eq!(sheet.rules[0].declarations.len(), 3);
    }
}
```

## Widget Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_widget_creation() {
        let button = Button::new("Test");
        assert_eq!(button.type_name(), "button");
        assert!(button.classes().contains("button"));
    }

    #[test]
    fn test_widget_builder() {
        let button = Button::new("Test")
            .class("primary")
            .id("my-button");

        assert!(button.classes().contains("primary"));
        assert_eq!(button.element_id(), Some("my-button"));
    }

    #[test]
    fn test_widget_state_changes() {
        let mut button = Button::new("Test");
        let mut ctx = EventContext::new();

        // Simulate hover
        let event = Event::Mouse(MouseEvent::new(
            MouseEventKind::Enter,
            Point::new(10.0, 10.0),
        ));
        button.handle_event(&event, &mut ctx);

        assert!(button.state().hovered);
    }
}
```

## Integration Tests

Place in `tests/` directory:

```rust
// tests/layout_integration.rs
use openkit::prelude::*;
use openkit::layout::*;

#[test]
fn test_column_layout() {
    let mut col = Column::new()
        .gap(8.0)
        .child(Label::new("One"))
        .child(Label::new("Two"));

    let constraints = Constraints::loose(Size::new(200.0, 400.0));
    let theme = ThemeData::light();
    let style_ctx = StyleContext::new(&theme);
    let layout_ctx = LayoutContext::new(&style_ctx);

    let result = col.layout(constraints, &layout_ctx);

    assert!(result.size.height > 0.0);
}
```

## Doc Tests

Add examples to documentation:

```rust
/// Creates a styled button.
///
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
///
/// let button = Button::new("Click me")
///     .class("primary")
///     .on_click(|| println!("Clicked!"));
///
/// assert_eq!(button.type_name(), "button");
/// ```
pub fn new(label: impl Into<String>) -> Self {
    // ...
}
```

For examples needing runtime:

```rust
/// # Examples
///
/// ```rust,ignore
/// App::new()
///     .title("My App")
///     .run(|| label!("Hello"));
/// ```
```

## Test Helpers

Create reusable test utilities:

```rust
// tests/helpers/mod.rs
pub fn create_test_theme() -> ThemeData {
    ThemeData::light()
}

pub fn create_test_context(theme: &ThemeData) -> StyleContext {
    StyleContext::new(theme)
}

pub fn create_test_widget<W: Widget>(widget: W) -> W {
    widget
}
```

## Running Tests

```bash
# All tests
cargo test

# Specific module
cargo test css::parser

# With output
cargo test -- --nocapture

# Doc tests only
cargo test --doc
```

## Checklist

When writing tests:

- [ ] Use descriptive test names (`test_<what>_<condition>_<expected>`)
- [ ] Follow Arrange-Act-Assert pattern
- [ ] Test both success and failure cases
- [ ] Test edge cases and boundary conditions
- [ ] Use `pretty_assertions` for better diffs
- [ ] Add doc tests for public API examples
- [ ] Keep tests focused and independent


## Cursor rule: `.cursor/agents/widget-builder.mdc`

_Agent for creating new widgets following OpenKit patterns_

# Widget Builder Agent

You are a specialized agent for creating new widgets in the OpenKit UI framework.

## Your Responsibilities

1. Create new widget implementations following the established patterns
2. Ensure widgets implement the `Widget` trait correctly
3. Add proper CSS class support
4. Implement event handling (mouse, keyboard)
5. Add builder methods for configuration
6. Create appropriate default styles

## Widget Template

When creating a new widget, follow this structure:

```rust
//! {WidgetName} widget.

use super::{Widget, WidgetBase, WidgetId, LayoutContext, PaintContext, EventContext};
use crate::css::{ClassList, ComputedStyle, StyleContext, WidgetState};
use crate::event::{Event, EventResult, MouseEvent, MouseEventKind, MouseButton};
use crate::geometry::{BorderRadius, Color, Point, Rect, Size};
use crate::layout::{Constraints, LayoutResult};
use crate::render::Painter;

/// A {description} widget.
pub struct {WidgetName} {
    base: WidgetBase,
    // Add widget-specific fields here
}

impl {WidgetName} {
    /// Create a new {WidgetName}.
    pub fn new() -> Self {
        Self {
            base: WidgetBase::new().with_class("{widget-name}"),
        }
    }

    /// Add a CSS class.
    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }

    /// Set the element ID.
    pub fn id(mut self, id: &str) -> Self {
        self.base.element_id = Some(id.to_string());
        self
    }
}

impl Widget for {WidgetName} {
    fn id(&self) -> WidgetId { self.base.id }
    fn type_name(&self) -> &'static str { "{widget-name}" }
    fn element_id(&self) -> Option<&str> { self.base.element_id.as_deref() }
    fn classes(&self) -> &ClassList { &self.base.classes }
    fn state(&self) -> WidgetState { self.base.state }

    fn intrinsic_size(&self, _ctx: &LayoutContext) -> Size {
        // Calculate intrinsic size
        Size::new(100.0, 40.0)
    }

    fn layout(&mut self, constraints: Constraints, ctx: &LayoutContext) -> LayoutResult {
        let intrinsic = self.intrinsic_size(ctx);
        let size = constraints.constrain(intrinsic);
        self.base.bounds.size = size;
        LayoutResult::new(size)
    }

    fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
        let theme = ctx.style_ctx.theme;
        // Implement painting
    }

    fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) -> EventResult {
        // Implement event handling
        EventResult::Ignored
    }

    fn bounds(&self) -> Rect { self.base.bounds }
    fn set_bounds(&mut self, bounds: Rect) { self.base.bounds = bounds; }
}
```

## Checklist

When creating a widget, ensure:

- [ ] `WidgetBase` is used for common properties
- [ ] Default CSS class matches widget name (kebab-case)
- [ ] Builder methods return `Self` for chaining
- [ ] `class()` and `id()` methods are implemented
- [ ] All `Widget` trait methods are implemented
- [ ] Event handling updates state and requests redraw
- [ ] Paint method uses theme colors
- [ ] Focus ring is drawn when focused

## File Location

New widgets go in `src/widget/{widget_name}.rs` and must be added to `src/widget/mod.rs`.


## Cursor rule: `.cursor/rules/component-system.mdc`

_Angular-like component system patterns_

Applies to: `["src/component.rs", "examples/angular_style.rs"]`

# Component System (Angular-like)

OpenKit provides an Angular-inspired component system for building reusable UI components.

## Defining Components

Use `define_component!` macro:

```rust
define_component!(
    CounterComponent,
    // State
    {
        count: State<i32>,
    },
    // Props
    {
        initial_value: i32,
        step: i32,
    },
    // Events
    {
        on_change: EventEmitter<i32>,
    },
    // Render
    |ctx| {
        let count = ctx.state.count.get();
        col![8;
            label!(format!("Count: {}", count)),
            row![4;
                button!("-", {
                    let new_val = count - ctx.props.step;
                    ctx.state.count.set(new_val);
                    ctx.events.on_change.emit(new_val);
                }),
                button!("+", {
                    let new_val = count + ctx.props.step;
                    ctx.state.count.set(new_val);
                    ctx.events.on_change.emit(new_val);
                }),
            ]
        ]
    }
);
```

## State Management

### `State<T>` - Reactive state
```rust
let count: State<i32> = State::new(0);

// Get value
let value = count.get();

// Set value (triggers re-render)
count.set(42);

// Update with function
count.update(|v| v + 1);
```

### `Model<T>` - Two-way binding
```rust
let name: Model<String> = Model::new(String::new());

// Bind to text field
textfield!("Name", model!(name))
```

## Event Emitters

```rust
let on_submit: EventEmitter<String> = EventEmitter::new();

// Emit event
on_submit.emit("submitted value".to_string());

// Subscribe to events
on_submit.subscribe(|value| {
    println!("Received: {}", value);
});
```

## Lifecycle Hooks

Components support lifecycle hooks:

```rust
impl Lifecycle for MyComponent {
    fn on_init(&mut self, ctx: &mut ComponentContext) {
        // Called when component initializes
    }

    fn on_changes(&mut self, changes: &Changes) {
        // Called when props change
    }

    fn on_render(&mut self, ctx: &mut ComponentContext) {
        // Called after each render
    }

    fn on_destroy(&mut self) {
        // Called when component is removed
    }
}
```

## Structural Directives

### `ng_if!` - Conditional rendering
```rust
ng_if!(show_message,
    label!("This is shown conditionally")
)
```

### `ng_for!` - List rendering
```rust
ng_for!(items, |item, index| {
    label!(format!("{}: {}", index, item.name))
})
```

### `ng_switch!` - Switch rendering
```rust
ng_switch!(status,
    "loading" => label!("Loading..."),
    "error" => label!("Error occurred"),
    "success" => label!("Success!"),
    _ => label!("Unknown state")
)
```

## Using Components

```rust
ComponentBuilder::new(CounterComponent::default())
    .prop("initial_value", 10)
    .prop("step", 5)
    .on("on_change", |value: i32| {
        println!("Counter changed to: {}", value);
    })
    .build()
```

## Best Practices

1. Keep components focused on a single responsibility
2. Use `State<T>` for internal component state
3. Use props for configuration passed from parent
4. Use events to communicate changes to parent
5. Implement lifecycle hooks only when needed
6. Prefer composition over inheritance


## Cursor rule: `.cursor/rules/css-styling.mdc`

_CSS styling system and conventions_

Applies to: `["src/css/**/*.rs", "**/*.css"]`

# CSS Styling System

## Supported CSS Features

### Selectors
- Type selectors: `button`, `label`
- Class selectors: `.btn-primary`
- ID selectors: `#my-widget`
- Pseudo-classes: `:hover`, `:active`, `:focus`, `:disabled`, `:checked`
- Universal selector: `*`

### Properties (Current Support)
- `background-color`
- `color`
- `padding` (all sides or individual)
- `margin`
- `border-width`, `border-color`, `border-radius`
- `font-size`, `font-weight`
- `width`, `height`, `min-width`, `max-width`
- `gap` (for flex containers)
- `flex-direction`, `justify-content`, `align-items`

### Units
- `px` - Pixels
- `rem` - Root em (relative to base font size)
- `em` - Em (relative to parent font size)
- `%` - Percentage
- `vw`, `vh` - Viewport units

## Theme System

The theme provides CSS variables (design tokens):

```css
:root {
  /* Colors */
  --background: #ffffff;
  --foreground: #0f172a;
  --primary: #3b82f6;
  --primary-foreground: #ffffff;
  --secondary: #f1f5f9;
  --muted: #f1f5f9;
  --muted-foreground: #64748b;
  --destructive: #ef4444;
  --border: #e2e8f0;
  --ring: #3b82f6;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;

  /* Typography */
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;

  /* Radii */
  --radius-sm: 0.125rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
}
```

## Loading Custom CSS

Use `StyleManager` to load custom CSS:

```rust
use openkit::css::StyleManager;

let mut styles = StyleManager::new();

// Load from file
styles.load_file("./custom.css")?;

// Load from string
styles.load_css(r#"
    .my-button {
        background-color: #3b82f6;
        border-radius: 8px;
    }
"#)?;

// Set CSS variables
styles.set_variable("--primary", "#8b5cf6");

// Use in app
App::new()
    .styles(styles)
    .run(|| { /* ... */ });
```

## Adding CSS Parser Support

When adding new CSS properties:

1. Add to `StyleProperty` enum in `src/css/properties.rs`
2. Add parsing logic in `src/css/parser.rs`
3. Add application logic in `ComputedStyle::apply()`
4. Update widget paint methods to use the property

## Default Styles

Framework default styles are in `src/css/default.css`. Keep this file simple with basic class definitions. Complex features like `:root` variables are handled programmatically by the theme system.


## Cursor rule: `.cursor/rules/macros-usage.mdc`

_Declarative UI macro patterns and usage_

Applies to: `["src/macros.rs", "examples/**/*.rs"]`

# Declarative UI Macros

OpenKit provides ergonomic macros for building UIs declaratively.

## Layout Macros

### `col!` - Vertical Column
```rust
col![gap;
    child1,
    child2,
    child3,
]

// Example
col![16;
    label!("Title"),
    button!("Click me", { println!("clicked"); }),
]
```

### `row!` - Horizontal Row
```rust
row![gap;
    child1,
    child2,
]

// Example
row![8;
    button!("OK", { /* ... */ }),
    button!("Cancel", Secondary, { /* ... */ }),
]
```

## Widget Macros

### `label!`
```rust
label!("Text content")
```

### `button!`
```rust
// Basic button with handler
button!("Label", { println!("clicked"); })

// With variant
button!("Delete", Destructive, { /* ... */ })

// Without handler
button!("Disabled")
```

### `checkbox!`
```rust
checkbox!("Label", |checked| {
    println!("Checked: {}", checked);
})

// With initial state
checkbox!("Accept", true, |checked| { /* ... */ })
```

### `textfield!`
```rust
textfield!("Placeholder...", |value| {
    println!("Value: {}", value);
})
```

## Utility Macros

### `spacer!` - Flexible space
```rust
row![8;
    label!("Left"),
    spacer!(),  // Pushes content to edges
    label!("Right"),
]
```

### `class!` - CSS class list
```rust
let classes = class!["btn", "primary", "large"];
```

### `style!` - Inline styles
```rust
let styles = style! {
    padding: 16,
    color: "blue"
};
```

### `when!` - Conditional widget
```rust
when!(condition,
    label!("Shown when true")
)
```

### `for_each!` - List rendering
```rust
for_each!(items, |item| {
    label!(item.name)
})
```

## Adding CSS Classes to Widgets

Use the `.class()` method after macro creation:

```rust
// Using builder method after macro
Label::new("Styled Text").class("hero-text")

Button::new("Custom Button")
    .class("gradient-btn")
    .on_click(|| println!("clicked"))
```

## Best Practices

1. Use macros for quick prototyping and simple UIs
2. Use builder pattern for complex widget configuration
3. Prefer `col!` and `row!` over manual container creation
4. Use `spacer!()` for flexible layouts
5. Keep handler closures short; extract logic to functions


## Cursor rule: `.cursor/rules/proc-macros.mdc`

_Procedural macro development patterns_

Applies to: `["openkit-macros/**/*.rs"]`

# Procedural Macros (openkit-macros)

The `openkit-macros` crate provides derive and attribute macros for OpenKit.

## Crate Structure

```
openkit-macros/
├── Cargo.toml
└── src/
    ├── lib.rs       # Macro entry points
    ├── widget.rs    # Widget derive implementation
    ├── component.rs # Component derive implementation
    └── styleable.rs # Styleable derive implementation
```

## Dependencies

- `proc-macro2` - TokenStream manipulation
- `quote` - Code generation
- `syn` - Parsing Rust code (v2.0)
- `darling` - Attribute parsing
- `proc-macro-error` - Better error messages

## Available Macros

### `#[derive(Widget)]`

Generates Widget trait boilerplate:

```rust
#[derive(Widget)]
#[widget(type_name = "my-widget")]
struct MyWidget {
    #[base]
    base: WidgetBase,
    label: String,
}
```

Generated:
- `widget_id()`, `widget_type_name()`, `widget_element_id()`
- `widget_classes()`, `widget_state()`
- `widget_bounds()`, `set_widget_bounds()`

### `#[derive(Component)]`

Generates Angular-like component infrastructure:

```rust
#[derive(Component)]
#[component(selector = "counter")]
struct Counter {
    #[state]
    count: i32,

    #[prop]
    step: i32,

    #[event]
    on_change: EventEmitter<i32>,
}
```

Generated:
- `CounterState` struct with `State<T>` wrappers
- `CounterProps` struct
- `CounterEvents` struct
- `new()` constructor
- `selector()` method

### `#[derive(Styleable)]`

Generates CSS styling methods:

```rust
#[derive(Styleable)]
struct MyWidget {
    #[base]
    base: WidgetBase,
}
```

Generated:
- `class(&str) -> Self`
- `classes(&[&str]) -> Self`
- `id(&str) -> Self`
- `remove_class()`, `toggle_class()`, `has_class()`

## Implementation Patterns

### Derive Macro Structure

```rust
#[proc_macro_derive(MacroName, attributes(attr1, attr2))]
#[proc_macro_error]
pub fn derive_macro(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    impl_macro(input)
        .unwrap_or_else(|e| e.to_compile_error())
        .into()
}

fn impl_macro(input: DeriveInput) -> Result<TokenStream, Error> {
    let name = &input.ident;

    // Parse attributes and fields
    // Generate code with quote!

    let expanded = quote! {
        impl #name {
            // generated code
        }
    };

    Ok(expanded)
}
```

### Attribute Parsing with darling

```rust
use darling::FromDeriveInput;

#[derive(FromDeriveInput)]
#[darling(attributes(widget))]
struct WidgetAttrs {
    ident: Ident,
    #[darling(default)]
    type_name: Option<String>,
}
```

### Field Iteration

```rust
let fields = match &input.data {
    Data::Struct(data) => &data.fields,
    _ => return Err(Error::new_spanned(input, "Expected struct")),
};

let named = match fields {
    Fields::Named(named) => named,
    _ => return Err(Error::new_spanned(input, "Expected named fields")),
};

for field in &named.named {
    let ident = field.ident.as_ref().unwrap();
    let ty = &field.ty;

    // Check for attributes
    let has_attr = field.attrs.iter().any(|a| a.path().is_ident("my_attr"));
}
```

### Code Generation with quote!

```rust
use quote::{quote, format_ident};

let method_name = format_ident!("get_{}", field_name);

let expanded = quote! {
    impl #struct_name {
        pub fn #method_name(&self) -> &#field_type {
            &self.#field_name
        }
    }
};
```

## Testing Proc Macros

Use `trybuild` for compile-fail tests:

```rust
#[test]
fn ui() {
    let t = trybuild::TestCases::new();
    t.pass("tests/pass/*.rs");
    t.compile_fail("tests/fail/*.rs");
}
```

## Best Practices

1. Use `#[proc_macro_error]` for better error messages
2. Return `syn::Error` for compile errors
3. Use `darling` for attribute parsing
4. Generate doc comments on generated code
5. Test with `trybuild` for compile-fail cases
6. Keep macro implementations in separate modules


## Cursor rule: `.cursor/rules/project-overview.mdc`

_OpenKit project overview and architecture_

Applies to: `["**/*.rs", "**/*.css"]`

# OpenKit Project Overview

OpenKit is a cross-platform CSS-styled UI framework for Rust. It provides consistent, beautiful desktop applications across Windows, macOS, and Linux with CSS-powered styling and a Tailwind-inspired design system.

## Core Architecture

```
Application Code
       ↓
Widget Tree (Rust)
       ↓
CSS Style Engine (Parser → Cascade → Computed → Layout)
       ↓
Flexbox/Grid Layout
       ↓
Render Layer (GPU via wgpu | Software via tiny-skia)
       ↓
Platform Window (Win32 | AppKit | GTK4/X11/Wayland)
```

## Key Principles

1. **Consistent Styling**: Apps look identical on Windows, macOS, and Linux
2. **CSS-Powered**: Style everything with familiar CSS syntax
3. **Tailwind by Default**: Ship with Tailwind-inspired design tokens
4. **GPU-First Rendering**: Custom-rendered widgets with GPU acceleration
5. **Accessible by Default**: Proper accessibility semantics built-in
6. **Zero JavaScript**: Pure Rust, no Electron, no WebView

## Module Structure

- `src/app.rs` - Application entry point and lifecycle
- `src/css/` - CSS parsing, styling, and cascade engine
- `src/widget/` - Widget implementations (Button, Label, TextField, etc.)
- `src/layout/` - Flexbox layout engine
- `src/render/` - GPU (wgpu) and CPU (tiny-skia) rendering backends
- `src/platform/` - Platform abstraction (winit-based)
- `src/theme/` - Tailwind-inspired theme system
- `src/component.rs` - Angular-like component system
- `src/macros.rs` - Declarative UI macros
- `src/geometry.rs` - Point, Size, Rect, Color types
- `src/event.rs` - Event types and handling

## Dependencies

- `winit` - Cross-platform windowing
- `wgpu` - GPU rendering (optional, default)
- `tiny-skia` - CPU rendering fallback
- `cosmic-text` - Text rendering
- `cssparser` - CSS parsing


## Cursor rule: `.cursor/rules/rendering.mdc`

_Rendering system patterns (GPU and CPU)_

Applies to: `["src/render/**/*.rs"]`

# Rendering System

OpenKit supports both GPU (wgpu) and CPU (tiny-skia) rendering backends.

## Renderer Architecture

```
Renderer (trait)
├── GpuRenderer (wgpu) - Default, GPU-accelerated
└── CpuRenderer (tiny-skia) - Software fallback
```

## Painter API

The `Painter` provides a unified drawing API:

```rust
pub struct Painter {
    commands: Vec<DrawCommand>,
}

impl Painter {
    // Rectangles
    pub fn fill_rect(&mut self, rect: Rect, color: Color);
    pub fn fill_rounded_rect(&mut self, rect: Rect, color: Color, radius: BorderRadius);
    pub fn stroke_rect(&mut self, rect: Rect, color: Color, width: f32);

    // Text
    pub fn draw_text(&mut self, text: &str, pos: Point, color: Color, size: f32);

    // Images
    pub fn draw_image(&mut self, image: &Image, rect: Rect);

    // Finish and get commands
    pub fn finish(self) -> Vec<DrawCommand>;
}
```

## Draw Commands

```rust
pub enum DrawCommand {
    FillRect { rect: Rect, color: Color },
    FillRoundedRect { rect: Rect, color: Color, radius: BorderRadius },
    StrokeRect { rect: Rect, color: Color, width: f32 },
    DrawText { text: String, position: Point, color: Color, size: f32 },
    DrawImage { rect: Rect, image_id: u64 },
    PushClip { rect: Rect },
    PopClip,
}
```

## Implementing Widget Paint

```rust
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    let theme = ctx.style_ctx.theme;

    // 1. Background
    painter.fill_rounded_rect(
        rect,
        self.background_color(theme),
        BorderRadius::all(theme.radii.md),
    );

    // 2. Border
    if self.variant == ButtonVariant::Outline {
        painter.stroke_rect(rect, theme.colors.border, 1.0);
    }

    // 3. Content
    let text_pos = Point::new(
        rect.x() + (rect.width() - text_width) / 2.0,
        rect.y() + (rect.height() + font_size) / 2.0,
    );
    painter.draw_text(&self.label, text_pos, text_color, font_size);

    // 4. Focus ring
    if self.base.state.focused && ctx.focus_visible {
        let ring_rect = rect.inflate(2.0, 2.0);
        painter.stroke_rect(ring_rect, theme.colors.ring, 2.0);
    }
}
```

## Color Utilities

```rust
impl Color {
    pub fn with_alpha(self, alpha: f32) -> Color;
    pub fn darken(self, percent: f32) -> Color;
    pub fn lighten(self, percent: f32) -> Color;
    pub fn from_hex(hex: &str) -> Result<Color, ColorError>;
}
```

## GPU vs CPU Rendering

The renderer is selected automatically:
- GPU renderer is used by default when available
- CPU renderer is used as fallback

Feature flag in `Cargo.toml`:
```toml
[features]
default = ["gpu"]
gpu = ["wgpu"]
```

Disable GPU for CPU-only:
```bash
cargo build --no-default-features
```

## Performance Tips

1. Minimize draw commands by batching similar operations
2. Use `PushClip`/`PopClip` for clipping instead of complex shapes
3. Cache computed styles when possible
4. Avoid unnecessary repaints - only call `ctx.request_redraw()` when needed


## Cursor rule: `.cursor/rules/rust-conventions.mdc`

_Rust coding conventions for OpenKit_

Applies to: `["**/*.rs"]`

# Rust Coding Conventions

## General Style

- Use Rust 2021 edition
- Follow standard Rust naming conventions (snake_case for functions/variables, PascalCase for types)
- Prefer `impl Into<T>` for string parameters: `fn new(label: impl Into<String>)`
- Use builder pattern for widget construction with method chaining
- Prefer `#[derive]` for common traits: Debug, Clone, Default, PartialEq

## Error Handling

- Use `thiserror` for custom error types
- Prefer `Result<T, E>` over panics in library code
- Use `log` crate for logging (warn, error, info, debug)
- Initialize logger with `env_logger::init()` in app entry

## Widget Implementation Pattern

Every widget should:
1. Have a `WidgetBase` field for common properties
2. Implement the `Widget` trait
3. Provide builder methods that return `Self`
4. Support CSS class assignment via `.class()` method

```rust
pub struct MyWidget {
    base: WidgetBase,
    // widget-specific fields
}

impl MyWidget {
    pub fn new() -> Self {
        Self {
            base: WidgetBase::new().with_class("my-widget"),
        }
    }

    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }
}

impl Widget for MyWidget {
    fn id(&self) -> WidgetId { self.base.id }
    fn type_name(&self) -> &'static str { "my-widget" }
    fn classes(&self) -> &ClassList { &self.base.classes }
    // ... implement other required methods
}
```

## Testing

- Use Vitest for any TypeScript/JavaScript (per user preference)
- Use `#[cfg(test)]` module for Rust tests
- Use `pretty_assertions` for better test diffs
- Mark doc examples with `rust,ignore` if they require runtime

## Imports

Organize imports in this order:
1. Standard library (`std::`)
2. External crates
3. Crate modules (`crate::`)
4. Super/self imports

```rust
use std::collections::HashMap;

use wgpu::Instance;

use crate::css::StyleContext;
use crate::widget::Widget;
```

## Documentation

- Add doc comments (`///`) to all public items
- Include examples in doc comments where helpful
- Use `//!` for module-level documentation


## Cursor rule: `.cursor/rules/testing.mdc`

_Testing conventions and patterns_

Applies to: `["**/*.rs", "**/*.ts", "**/*.js"]`

# Testing Conventions

## Rust Testing

### Unit Tests
Place unit tests in a `#[cfg(test)]` module at the bottom of each file:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use pretty_assertions::assert_eq;

    #[test]
    fn test_widget_creation() {
        let btn = Button::new("Test");
        assert_eq!(btn.type_name(), "button");
    }

    #[test]
    fn test_css_parsing() {
        let result = CssParser::parse_stylesheet(".test { color: red; }");
        assert!(result.is_ok());
    }
}
```

### Integration Tests
Place integration tests in `tests/` directory:

```
tests/
├── css_integration.rs
├── layout_integration.rs
└── widget_integration.rs
```

### Doc Tests
Add testable examples to documentation:

```rust
/// Creates a new button widget.
///
/// # Examples
///
/// ```rust
/// use openkit::prelude::*;
///
/// let button = Button::new("Click me")
///     .class("primary")
///     .on_click(|| println!("clicked"));
/// ```
pub fn new(label: impl Into<String>) -> Self {
    // ...
}
```

Mark examples that need runtime with `ignore`:

```rust
/// ```rust,ignore
/// App::new().run(|| { /* ... */ });
/// ```
```

## TypeScript/JavaScript Testing

Use **Vitest** for all TypeScript/JavaScript tests (per user preference):

```typescript
import { describe, it, expect } from 'vitest';

describe('Component', () => {
    it('should render correctly', () => {
        // test implementation
    });
});
```

## Test Organization

```
src/
├── css/
│   ├── mod.rs
│   ├── parser.rs      # Unit tests at bottom
│   └── selector.rs    # Unit tests at bottom
├── widget/
│   └── button.rs      # Unit tests at bottom
tests/
├── css_integration.rs
└── widget_integration.rs
```

## Running Tests

```bash
# Run all tests
cargo test

# Run specific test module
cargo test css::parser

# Run with output
cargo test -- --nocapture

# Run ignored tests
cargo test -- --ignored
```

## Test Fixtures

For CSS testing, use inline CSS strings:

```rust
#[test]
fn test_css_cascade() {
    let css = r#"
        .base { color: red; }
        .override { color: blue; }
    "#;

    let sheet = CssParser::parse_stylesheet(css).unwrap();
    // assertions...
}
```

## Assertions

Use `pretty_assertions` for better diff output:

```rust
use pretty_assertions::{assert_eq, assert_ne};

#[test]
fn test_computed_style() {
    let expected = ComputedStyle {
        color: Color::RED,
        // ...
    };
    let actual = compute_style(&widget, &ctx);
    assert_eq!(actual, expected);
}
```


## Cursor rule: `.cursor/rules/widget-development.mdc`

_Widget development patterns and best practices_

Applies to: `["src/widget/**/*.rs"]`

# Widget Development

## Widget Trait Requirements

All widgets must implement the `Widget` trait:

```rust
pub trait Widget {
    fn id(&self) -> WidgetId;
    fn type_name(&self) -> &'static str;
    fn element_id(&self) -> Option<&str>;
    fn classes(&self) -> &ClassList;
    fn state(&self) -> WidgetState;
    fn intrinsic_size(&self, ctx: &LayoutContext) -> Size;
    fn layout(&mut self, constraints: Constraints, ctx: &LayoutContext) -> LayoutResult;
    fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext);
    fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) -> EventResult;
    fn bounds(&self) -> Rect;
    fn set_bounds(&mut self, bounds: Rect);
}
```

## Widget State Management

Use `WidgetState` for tracking interactive states:

```rust
pub struct WidgetState {
    pub hovered: bool,
    pub pressed: bool,
    pub focused: bool,
    pub disabled: bool,
    pub checked: bool,  // For checkboxes, toggles
    pub first_child: bool,
    pub last_child: bool,
    pub nth_child: usize,
}
```

## Event Handling Pattern

```rust
fn handle_event(&mut self, event: &Event, ctx: &mut EventContext) -> EventResult {
    match event {
        Event::Mouse(mouse) => {
            let in_bounds = self.bounds().contains(mouse.position);

            match mouse.kind {
                MouseEventKind::Enter | MouseEventKind::Move => {
                    if in_bounds && !self.base.state.hovered {
                        self.base.state.hovered = true;
                        ctx.request_redraw();
                    }
                }
                MouseEventKind::Down if in_bounds => {
                    self.base.state.pressed = true;
                    ctx.request_focus(self.base.id);
                    ctx.request_redraw();
                    return EventResult::Handled;
                }
                // ... handle other events
                _ => {}
            }
        }
        _ => {}
    }
    EventResult::Ignored
}
```

## Painting Pattern

```rust
fn paint(&self, painter: &mut Painter, rect: Rect, ctx: &PaintContext) {
    let theme = ctx.style_ctx.theme;

    // 1. Draw background
    let bg_color = self.background_color(theme);
    let radius = BorderRadius::all(theme.radii.md);
    painter.fill_rounded_rect(rect, bg_color, radius);

    // 2. Draw border (if applicable)
    if self.has_border() {
        painter.stroke_rect(rect, theme.colors.border, 1.0);
    }

    // 3. Draw content (text, icons, etc.)
    painter.draw_text(&self.label, position, text_color, font_size);

    // 4. Draw focus ring (if focused)
    if self.base.state.focused && ctx.focus_visible {
        painter.stroke_rect(focus_rect, theme.colors.ring, 2.0);
    }
}
```

## CSS Class Support

Always support CSS class assignment:

```rust
impl MyWidget {
    pub fn class(mut self, class: &str) -> Self {
        self.base.classes.add(class);
        self
    }

    pub fn id(mut self, id: &str) -> Self {
        self.base.element_id = Some(id.to_string());
        self
    }
}
```

## Container Widgets

For widgets that contain children:

```rust
pub struct Container {
    base: WidgetBase,
    children: Vec<Box<dyn Widget>>,
    gap: f32,
}

impl Widget for Container {
    fn children(&self) -> &[Box<dyn Widget>] {
        &self.children
    }

    fn children_mut(&mut self) -> &mut [Box<dyn Widget>] {
        &mut self.children
    }
}
```

