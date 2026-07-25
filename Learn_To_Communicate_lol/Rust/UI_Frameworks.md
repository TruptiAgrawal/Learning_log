# Learning Log: Rust UI Packages
## 1. `ratatui` — Terminal User Interface

**What it is**
A library for building rich, interactive UIs that run entirely inside a terminal emulator. It is a maintained fork of `tui-rs`. It renders to a virtual buffer and diffs against the previous frame before writing to stdout — so only changed cells get redrawn.

**Mental model**
Immediate-mode, but for the terminal. Each "frame" you describe the whole layout using composable widgets; ratatui figures out what changed and flushes it. You own the event loop and drive the `Terminal::draw` call yourself.

**Core concepts**
- `Terminal<B>` — wraps a backend (e.g. `CrosstermBackend`) and owns the frame buffer
- `Frame` — handed to your draw closure; you call `render_widget(widget, area)` on it
- `Rect` — the fundamental layout primitive; everything is a rectangle of terminal cells
- Widgets — `Paragraph`, `Block`, `Table`, `List`, `Chart`, `Gauge`, `Canvas`, etc. all implement `Widget` and are composable
- Layout — `Layout::default().constraints([...]).split(area)` divides a `Rect` into sub-rects
- Crossterm — ratatui re-exports crossterm for raw mode, alternate screen, and key events

**Key API pattern**
```rust
terminal.draw(|f| {
    let para = Paragraph::new("hello")
        .block(Block::default().borders(Borders::ALL).title("box"));
    f.render_widget(para, f.area());
})?;
```

**Strengths**
- Zero external window dependencies — works over SSH, in Docker, CI output, headless servers
- Tiny binary footprint
- Extremely fast — terminal cells are cheap; redraw latency is microseconds
- Great for developer tools, CLIs, dashboards, log viewers, and anything ops-facing
- Works on every platform with a terminal

**Limitations**
- Fixed grid of cells — no sub-cell pixel positioning, no images (without extensions like Sixel)
- Font rendering is at the mercy of the user's terminal emulator
- No mouse support beyond basic click/scroll (crossterm provides it but it's coarse)
- Not suitable for consumer-facing apps or anything requiring rich media

**Best use cases**
- Developer tooling (TUI test runners, git UIs like `gitui`, package managers)
- Monitoring dashboards (metrics, log tails)
- Interactive CLI wizards
- Any tool where the user is already in a terminal

---

## 2. `eframe` / `egui` — Immediate-Mode Desktop GUI
**What it is**
`egui` is a pure-Rust, immediate-mode GUI library. `eframe` is the application harness that wraps it — it handles window creation, OS event loop, and the rendering backend (wgpu or glow). You interact almost exclusively with `egui`; `eframe` is just the launcher.

**Mental model**
Immediate-mode means **there is no widget state stored by the library**. Every frame, your `update()` function runs and describes the entire UI from scratch. If you call `ui.button("Click me")` and the user clicked it, that call returns `true` — and that's the entire event model. No callbacks, no observers, no widget IDs to manage.

Compare to retained-mode (most GUI frameworks): you create a Button object, attach a listener, and the framework calls your listener when the event fires. In immediate-mode, the "creation" and "event check" happen in the same line, every frame.

**Core concepts**
- `eframe::App` trait — implement `update(&mut self, ctx, frame)` for your app loop
- `egui::Context` — the root; you ask it for panels/windows
- `egui::Ui` — the drawing context passed into closures; all widget calls go here
- Panels — `CentralPanel`, `SidePanel`, `TopBottomPanel` — subdivide the window
- Layout — `ui.horizontal(|ui| ...)`, `ui.vertical(|ui| ...)`, `ui.columns(n, |cols| ...)`
- Painter — `ui.allocate_painter(size, sense)` gives you a `Painter` for raw 2D drawing (rects, lines, text, circles)
- Response — every widget call returns a `Response` that has `.clicked()`, `.hovered()`, `.drag_delta()`, etc.

**Key API pattern**
```rust
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        ui.heading("Hello");
        if ui.button("Solve").clicked() {
            self.solve();
        }
        ui.add(egui::Slider::new(&mut self.speed, 1.0..=60.0));
    });
}
```

**Rendering pipeline**
egui emits a list of paint commands (triangles + textures). eframe feeds those to wgpu (or glow), which renders to the native window. This means you get GPU acceleration without managing any GPU code yourself.

**Strengths**
- Extremely fast to prototype — describe the UI, run it, see it
- No XML/CSS/DSL — the UI *is* your Rust code, so refactoring is easy
- `Painter` API makes custom 2D rendering trivial
- Good performance; 60fps is easy for complex UIs
- Runs on desktop (Windows/Mac/Linux) and in the browser via WASM
- Active ecosystem — `egui_extras`, `egui_plot`, `egui_dock`

**Limitations**
- Immediate-mode requires re-running all layout logic every frame — subtle inefficiency at extreme scale (thousands of widgets)
- Styling is limited compared to CSS-based toolkits; custom theming requires overriding `Visuals` structs
- No native widget look — draws everything itself, so it never looks like a "native" OS dialog
- Accessibility (screen readers, ARIA) is minimal
- Font control is coarse; simulating bold requires drawing text twice with an offset

**Best use cases**
- Game tooling, level editors, debuggers, profilers
- Internal/developer-facing desktop apps
- Visualizers and data exploration tools
- Rapid prototyping of any desktop GUI
- Tools that need to embed in a game engine (egui is used by Bevy)

---

## 3. `slint` — Declarative Desktop GUI
**What it is**
Slint is a declarative UI framework with its own compiled DSL (`.slint` files). The UI is defined in markup; Rust (or C++/JavaScript) provides the data and logic. It compiles the `.slint` file at build time via `slint-build` and generates Rust types you interact with.

**Mental model**
Retained-mode and data-driven. You define the UI structure and bindings once in `.slint`. When your Rust code changes a property on the window handle, Slint re-evaluates only the expressions that depend on that property — similar to how a spreadsheet recalculates dependent cells. You don't call a draw function every frame; you push data and the framework handles rendering.

**Core concepts**
- `.slint` file — defines components (`component MainWindow inherits Window { ... }`), their properties, and layout. Compiled to Rust at build time.
- `in property` / `in-out property` — typed properties Rust can set; `in-out` means both sides can read/write
- `callback` — a function signature declared in `.slint` that Rust implements via `window.on_<callback>(|args| { ... })`
- `for item in model : Component { ... }` — reactive list rendering; updating the model auto-rerenders
- `ModelRc<T>` / `VecModel<T>` — Slint's typed model wrappers for list data
- `slint::Timer` — a recurring or one-shot timer that fires closures on the UI thread
- `Weak<T>` handles — window handles are reference-counted; `.as_weak()` is cloned into closures to avoid cycles

**Key API pattern**
```slint
// In main.slint
in property <string> status-text;
callback play-clicked();
Button { text: "Play"; clicked => { root.play-clicked(); } }
```
```rust
// In main.rs
window.set_status_text("Solving…".into());
window.on_play_clicked(move || { /* Rust logic */ });
```

**Build integration**
`build.rs` runs `slint_build::compile("ui/main.slint")`, which parses the DSL and generates a Rust module. `slint::include_modules!()` in `main.rs` brings those generated types into scope. This means `.slint` syntax errors are compile errors.

**Modelling custom canvas-like rendering**
Because Slint's layout system is declarative (not a painter canvas), complex custom rendering requires flattening draw commands into a model. In `rust_v3`, the board is a `Vec<VisualItem>` — each cell and inequality symbol becomes a positioned rectangle with color/text properties — and the `.slint` file iterates over it with `for item in board-items`, rendering each as an absolutely-positioned `Rectangle`. This is the standard Slint pattern for custom rendering.

**Strengths**
- Clean separation: UI structure/style in `.slint`, logic in Rust — no UI strings scattered in Rust code
- Compile-time validation of the DSL — typos in property names are compiler errors
- Reactive bindings — changing a property automatically updates all dependent expressions without manual invalidation
- Designed for **embedded targets** (microcontrollers, automotive dashboards) as well as desktop — very small runtime, no heap allocator required in embedded mode
- Better accessibility groundwork compared to egui

**Limitations**
- The extra build step (`slint-build`) and generated code add complexity
- `for item in model` is less flexible than a Painter for complex custom rendering (requires the `VisualItem` flattening workaround)
- Smaller ecosystem than egui; fewer ready-made complex widgets
- Crossing the Rust↔Slint boundary for every state change (window setters) adds boilerplate
- The DSL is a new language to learn on top of Rust

**Best use cases**
- Embedded / IoT UIs (Slint's primary market)
- Desktop apps where you want a designer-friendly, CSS-like markup workflow
- Apps targeting both desktop and embedded from a shared UI description
- Teams where UI designers and Rust developers split work across `.slint` and `.rs` files

---

## Side-by-Side Comparison

| Dimension             | ratatui            | eframe/egui            | Slint                    |
|-----------------------|--------------------|------------------------|--------------------------|
| Rendering target      | Terminal cells     | GPU (wgpu/glow)        | GPU / embedded display   |
| UI paradigm           | Immediate-mode     | Immediate-mode         | Retained / declarative   |
| UI definition         | Rust code          | Rust code              | `.slint` DSL + Rust      |
| Custom drawing        | Character grid     | `Painter` 2D API       | Positioned model items   |
| Event model           | Poll crossterm     | `Response.clicked()`   | Typed callbacks + bindings |
| Embedded targets      | No                 | No                     | Yes (core use case)      |
| WASM / browser        | No                 | Yes                    | Yes                      |
| Learning curve        | Low                | Low–Medium             | Medium (two languages)   |
| Ecosystem maturity    | High               | High                   | Growing                  |
| Native OS look        | No (terminal)      | No (custom draw)       | No (custom draw)         |

---

## When to reach for each

- **ratatui** — your users are already in a terminal, or you're building a developer/ops tool where zero-dependency deployment matters.
- **egui** — you want a native desktop window, need to prototype fast, or your UI involves custom 2D rendering (visualizers, editors, debuggers).
- **Slint** — you're targeting embedded hardware alongside desktop, you want a clean markup/code split, or your team has designers who prefer a declarative UI language.
