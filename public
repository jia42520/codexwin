import ctypes
import os
import re
import shutil
import subprocess
from datetime import datetime
from pathlib import Path
import tkinter as tk
from tkinter import filedialog, messagebox

APP_TITLE = "Codex Win 免登录配置工具"
DEFAULT_CONFIG_PATH = Path.home() / ".codex" / "config.toml"
DEFAULT_AUTH_PATH = Path.home() / ".codex" / "auth.json"
FONT_FAMILY = "Microsoft YaHei UI"
FONT_TITLE = (FONT_FAMILY, 21, "bold")
FONT_BODY = (FONT_FAMILY, 10)
FONT_LABEL = (FONT_FAMILY, 10, "bold")
FONT_BUTTON = (FONT_FAMILY, 11, "bold")
FONT_SMALL = (FONT_FAMILY, 9)
RESERVED_PROVIDER_IDS = {"openai", "azure", "gemini", "ollama", "mistral", "openrouter", "xai", "groq"}


def enable_high_dpi() -> None:
    try:
        ctypes.windll.shcore.SetProcessDpiAwareness(2)
    except (AttributeError, OSError):
        try:
            ctypes.windll.user32.SetProcessDPIAware()
        except (AttributeError, OSError):
            pass


def escape_toml_string(value: str) -> str:
    return value.replace("\\", "\\\\").replace('"', '\\"')


def normalize_single_line(value: str) -> str:
    return "".join(value.strip().splitlines()).strip()


def normalize_provider_name(value: str) -> str:
    normalized = re.sub(r"[^A-Za-z0-9_-]", "_", normalize_single_line(value))
    normalized = normalized.strip("_") or "custom"
    if normalized.lower() in RESERVED_PROVIDER_IDS:
        normalized = f"{normalized}-custom"
    return normalized


def normalize_base_url(value: str) -> str:
    normalized = normalize_single_line(value).rstrip("/")
    for suffix in ("/chat/completions", "/responses"):
        if normalized.endswith(suffix):
            normalized = normalized[: -len(suffix)]
    return normalized.rstrip("/")


def build_config(provider_name: str, api_key: str, base_url: str, model: str) -> str:
    safe_provider = escape_toml_string(normalize_provider_name(provider_name))
    safe_api_key = escape_toml_string(normalize_single_line(api_key))
    safe_base_url = escape_toml_string(normalize_base_url(base_url))
    safe_model = escape_toml_string(normalize_single_line(model))

    return f'''model = "{safe_model}"
model_provider = "{safe_provider}"
preferred_auth_method = "apikey"

[model_providers.{safe_provider}]
name = "{safe_provider}"
base_url = "{safe_base_url}"
env_key = "CODEX_API_KEY"
wire_api = "responses"

[env]
CODEX_API_KEY = "{safe_api_key}"
'''


def backup_existing_config(config_path: Path) -> Path | None:
    if not config_path.exists():
        return None

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_path = config_path.with_suffix(config_path.suffix + f".bak_{timestamp}")
    shutil.copy2(config_path, backup_path)
    return backup_path


def disable_login_auth(auth_path: Path) -> Path | None:
    if not auth_path.exists():
        return None

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_path = auth_path.with_suffix(auth_path.suffix + f".bak_{timestamp}")
    shutil.move(str(auth_path), str(backup_path))
    return backup_path


def set_user_api_key(api_key: str) -> None:
    clean_api_key = normalize_single_line(api_key)
    subprocess.run(["setx", "CODEX_API_KEY", clean_api_key], check=True, capture_output=True, text=True)
    os.environ["CODEX_API_KEY"] = clean_api_key


def draw_rounded_rect(canvas: tk.Canvas, x1: int, y1: int, x2: int, y2: int, radius: int, **kwargs) -> None:
    points = [
        x1 + radius,
        y1,
        x2 - radius,
        y1,
        x2,
        y1,
        x2,
        y1 + radius,
        x2,
        y2 - radius,
        x2,
        y2,
        x2 - radius,
        y2,
        x1 + radius,
        y2,
        x1,
        y2,
        x1,
        y2 - radius,
        x1,
        y1 + radius,
        x1,
        y1,
    ]
    canvas.create_polygon(points, smooth=True, **kwargs)


def create_rounded_frame(parent: tk.Widget, bg: str, radius: int = 16, border: str | None = None) -> tuple[tk.Canvas, tk.Frame]:
    canvas = tk.Canvas(parent, height=300, bg=parent.cget("bg"), highlightthickness=0)
    canvas.pack(fill=tk.X)
    frame = tk.Frame(canvas, bg=bg, padx=18, pady=18)
    window_id = canvas.create_window(0, 0, anchor="nw", window=frame)

    def redraw(width: int, height: int) -> None:
        canvas.delete("rounded_bg")
        draw_rounded_rect(canvas, 1, 1, width - 1, height - 1, radius, fill=bg, outline=border or bg, tags="rounded_bg")
        canvas.tag_lower("rounded_bg")

    def resize_canvas(event: tk.Event) -> None:
        canvas.itemconfigure(window_id, width=event.width)
        redraw(event.width, int(canvas.cget("height")))

    def resize_frame(event: tk.Event) -> None:
        height = event.height + 8
        canvas.configure(height=height)
        redraw(canvas.winfo_width() or 1, height)

    canvas.bind("<Configure>", resize_canvas)
    frame.bind("<Configure>", resize_frame)
    return canvas, frame


class RoundedButton(tk.Canvas):
    def __init__(
        self,
        parent: tk.Widget,
        text: str,
        command,
        bg: str,
        hover_bg: str,
        fg: str,
        width: int,
        height: int = 42,
        radius: int = 18,
        font=FONT_BUTTON,
    ) -> None:
        super().__init__(parent, width=width, height=height, bg=parent.cget("bg"), highlightthickness=0, cursor="hand2")
        self.command = command
        self.normal_bg = bg
        self.hover_bg = hover_bg
        self.fg = fg
        self.radius = radius
        self.font = font
        self.text = text
        self.bind("<Button-1>", lambda _: self.command())
        self.bind("<Enter>", lambda _: self.draw(self.hover_bg))
        self.bind("<Leave>", lambda _: self.draw(self.normal_bg))
        self.draw(self.normal_bg)

    def draw(self, fill: str) -> None:
        self.delete("all")
        width = int(self.cget("width"))
        height = int(self.cget("height"))
        draw_rounded_rect(self, 1, 1, width - 1, height - 1, self.radius, fill=fill, outline=fill)
        self.create_text(width // 2, height // 2, text=self.text, fill=self.fg, font=self.font)


class CodexConfigApp(tk.Tk):
    def __init__(self) -> None:
        super().__init__()
        self.title(APP_TITLE)
        self.geometry("860x760")
        self.minsize(860, 760)
        self.resizable(True, True)
        self.configure(bg="#f3f6fb")

        self.config_path_var = tk.StringVar(value=str(DEFAULT_CONFIG_PATH))
        self.provider_var = tk.StringVar(value="lightconeapi")
        self.base_url_var = tk.StringVar(value="https://api.lightcone.hk/v1/chat/completions")
        self.model_var = tk.StringVar(value="gpt-5.5")
        self.api_key_var = tk.StringVar()
        self.remove_login_var = tk.BooleanVar(value=True)
        self.write_env_var = tk.BooleanVar(value=True)
        self.show_key_var = tk.BooleanVar(value=False)
        self.status_var = tk.StringVar(value="填写 API 信息后点击保存，会同时写入 Windows 用户环境变量。")

        self.create_widgets()

    def create_widgets(self) -> None:
        container = tk.Frame(self, padx=24, pady=16, bg="#f3f6fb")
        container.pack(fill=tk.BOTH, expand=True)

        header = tk.Frame(container, bg="#f3f6fb")
        header.pack(fill=tk.X)

        title = tk.Label(
            header,
            text="Codex 免登录配置",
            font=FONT_TITLE,
            bg="#f3f6fb",
            fg="#111827",
        )
        title.pack(anchor="w")

        subtitle = tk.Label(
            header,
            text="填写 API Key 后一键写入配置和环境变量。",
            font=FONT_BODY,
            bg="#f3f6fb",
            fg="#64748b",
        )
        subtitle.pack(anchor="w", pady=(4, 12))

        _, form = create_rounded_frame(container, "white", radius=18, border="#dbe4f0")

        self.add_path_row(form, 0, "配置路径", self.config_path_var)
        self.add_entry_row(form, 1, "Provider", self.provider_var)
        self.add_entry_row(form, 2, "API 地址", self.base_url_var)
        self.add_entry_row(form, 3, "模型", self.model_var)
        self.add_api_key_row(form, 4)
        self.add_env_options_row(form, 5)

        button_frame = tk.Frame(container, bg="#f3f6fb")
        button_frame.pack(fill=tk.X, pady=(14, 8))

        save_button = RoundedButton(
            button_frame,
            text="保存并启用",
            command=self.save_config,
            bg="#2563eb",
            hover_bg="#1d4ed8",
            fg="white",
            width=600,
            height=38,
            radius=18,
        )
        save_button.pack(side=tk.LEFT, fill=tk.X, expand=True)

        open_button = RoundedButton(
            button_frame,
            text="打开目录",
            command=self.open_config_dir,
            bg="#e2e8f0",
            hover_bg="#cbd5e1",
            fg="#334155",
            width=120,
            height=38,
            radius=18,
            font=FONT_BODY,
        )
        open_button.pack(side=tk.LEFT, padx=(12, 0))

        preview_label = tk.Label(
            container,
            text="配置预览",
            font=FONT_LABEL,
            bg="#f3f6fb",
            fg="#334155",
        )
        preview_label.pack(anchor="w", pady=(8, 6))

        self.preview_frame = tk.Frame(container, bg="#0f172a", padx=0, pady=0)
        self.preview_frame.pack(fill=tk.BOTH, expand=True)

        self.preview_scrollbar = tk.Scrollbar(self.preview_frame)
        self.preview_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.preview = tk.Text(
            self.preview_frame,
            height=10,
            wrap=tk.NONE,
            bg="#0f172a",
            fg="#e2e8f0",
            insertbackground="#e2e8f0",
            relief=tk.FLAT,
            padx=10,
            pady=8,
            font=("Consolas", 9),
            yscrollcommand=self.preview_scrollbar.set,
        )
        self.preview.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        self.preview_scrollbar.configure(command=self.preview.yview)
        self.preview.configure(state=tk.DISABLED)

        status = tk.Label(
            container,
            textvariable=self.status_var,
            anchor="w",
            bg="#f3f6fb",
            fg="#166534",
            font=FONT_SMALL,
        )
        status.pack(fill=tk.X, pady=(8, 0))

        for var in [self.provider_var, self.base_url_var, self.model_var, self.api_key_var]:
            var.trace_add("write", lambda *_: self.update_preview())
        self.update_preview()

    def add_path_row(self, parent: tk.Frame, row: int, label: str, variable: tk.StringVar) -> None:
        tk.Label(parent, text=label, width=12, anchor="w", bg="white", fg="#334155", font=FONT_LABEL).grid(row=row, column=0, sticky="w", pady=6)
        tk.Entry(parent, textvariable=variable, relief=tk.FLAT, bg="#f8fafc", highlightthickness=1, highlightbackground="#cbd5e1", font=FONT_BODY).grid(row=row, column=1, sticky="ew", ipady=5, pady=6)
        tk.Button(parent, text="选择", command=self.choose_config_path, relief=tk.FLAT, cursor="hand2", bg="#e2e8f0", activebackground="#cbd5e1", font=FONT_SMALL).grid(row=row, column=2, padx=(8, 0), ipadx=8, ipady=3, pady=6)
        parent.columnconfigure(1, weight=1)

    def add_entry_row(self, parent: tk.Frame, row: int, label: str, variable: tk.StringVar) -> None:
        tk.Label(parent, text=label, width=12, anchor="w", bg="white", fg="#334155", font=FONT_LABEL).grid(row=row, column=0, sticky="w", pady=6)
        tk.Entry(parent, textvariable=variable, relief=tk.FLAT, bg="#f8fafc", highlightthickness=1, highlightbackground="#cbd5e1", font=FONT_BODY).grid(row=row, column=1, columnspan=2, sticky="ew", ipady=5, pady=6)

    def add_api_key_row(self, parent: tk.Frame, row: int) -> None:
        tk.Label(parent, text="API Key", width=12, anchor="w", bg="white", fg="#334155", font=FONT_LABEL).grid(row=row, column=0, sticky="w", pady=6)
        self.api_key_entry = tk.Entry(parent, textvariable=self.api_key_var, show="*", relief=tk.FLAT, bg="#f8fafc", highlightthickness=1, highlightbackground="#cbd5e1", font=FONT_BODY)
        self.api_key_entry.grid(row=row, column=1, sticky="ew", ipady=5, pady=6)
        tk.Checkbutton(parent, text="显示", variable=self.show_key_var, command=self.toggle_api_key, bg="white", activebackground="white", font=FONT_SMALL).grid(
            row=row, column=2, padx=(8, 0), pady=6
        )

    def add_env_options_row(self, parent: tk.Frame, row: int) -> None:
        tk.Label(parent, text="选项", width=12, anchor="w", bg="white", fg="#334155", font=FONT_LABEL).grid(row=row, column=0, sticky="w", pady=6)
        options = tk.Frame(parent, bg="white")
        options.grid(row=row, column=1, columnspan=2, sticky="w", pady=6)
        tk.Checkbutton(
            options,
            text="写入环境变量 CODEX_API_KEY",
            variable=self.write_env_var,
            bg="white",
            activebackground="white",
            font=FONT_SMALL,
        ).pack(side=tk.LEFT)
        tk.Checkbutton(
            options,
            text="移除网页登录 auth.json",
            variable=self.remove_login_var,
            bg="white",
            activebackground="white",
            font=FONT_SMALL,
        ).pack(side=tk.LEFT, padx=(22, 0))

    def choose_config_path(self) -> None:
        path = filedialog.asksaveasfilename(
            title="选择 Codex 配置文件",
            initialdir=str(DEFAULT_CONFIG_PATH.parent),
            initialfile="config.toml",
            defaultextension=".toml",
            filetypes=[("TOML 配置文件", "*.toml"), ("所有文件", "*.*")],
        )
        if path:
            self.config_path_var.set(path)

    def toggle_api_key(self) -> None:
        self.api_key_entry.configure(show="" if self.show_key_var.get() else "*")

    def update_preview(self) -> None:
        api_key = self.api_key_var.get().strip()
        masked_key = "" if not api_key else api_key[:4] + "***" + api_key[-4:] if len(api_key) > 8 else "***"
        preview = build_config(
            self.provider_var.get(),
            masked_key,
            self.base_url_var.get(),
            self.model_var.get(),
        )
        self.preview.configure(state=tk.NORMAL)
        self.preview.delete("1.0", tk.END)
        self.preview.insert(tk.END, preview)
        self.preview.configure(state=tk.DISABLED)

    def validate_inputs(self) -> bool:
        required = [
            ("配置文件路径", self.config_path_var.get()),
            ("Provider 名称", self.provider_var.get()),
            ("API Base URL", self.base_url_var.get()),
            ("模型名称", self.model_var.get()),
            ("API Key", self.api_key_var.get()),
        ]
        missing = [name for name, value in required if not value.strip()]
        if missing:
            messagebox.showwarning("信息不完整", "请填写：" + "、".join(missing))
            return False
        if not normalize_base_url(self.base_url_var.get()).startswith(("http://", "https://")):
            messagebox.showwarning("地址格式错误", "API Base URL 需要以 http:// 或 https:// 开头。")
            return False
        self.provider_var.set(normalize_provider_name(self.provider_var.get()))
        self.base_url_var.set(normalize_base_url(self.base_url_var.get()))
        self.model_var.set(normalize_single_line(self.model_var.get()))
        self.api_key_var.set(normalize_single_line(self.api_key_var.get()))
        return True

    def save_config(self) -> None:
        if not self.validate_inputs():
            return

        config_path = Path(self.config_path_var.get()).expanduser()
        config_text = build_config(
            self.provider_var.get(),
            self.api_key_var.get(),
            self.base_url_var.get(),
            self.model_var.get(),
        )

        try:
            config_path.parent.mkdir(parents=True, exist_ok=True)
            backup_path = backup_existing_config(config_path)
            config_path.write_text(config_text, encoding="utf-8")
            if self.write_env_var.get():
                set_user_api_key(self.api_key_var.get())
            auth_backup_path = disable_login_auth(DEFAULT_AUTH_PATH) if self.remove_login_var.get() else None
        except (OSError, subprocess.CalledProcessError) as exc:
            messagebox.showerror("保存失败", f"写入配置或环境变量失败：\n{exc}")
            self.status_var.set("保存失败，请检查路径权限或环境变量权限。")
            return

        message_parts = [f"配置已保存：\n{config_path}"]
        if self.write_env_var.get():
            message_parts.append("已写入 Windows 用户环境变量 CODEX_API_KEY，新打开的终端会自动生效。")
        if backup_path:
            message_parts.append(f"旧配置已备份：\n{backup_path}")
        if auth_backup_path:
            message_parts.append(f"网页登录认证已移除并备份：\n{auth_backup_path}")
        elif self.remove_login_var.get():
            message_parts.append("未发现 auth.json，本机当前没有可移除的网页登录认证文件。")
        messagebox.showinfo("免登录已启用", "\n\n".join(message_parts))
        self.status_var.set("保存成功，配置和 CODEX_API_KEY 环境变量已写入。")

    def open_config_dir(self) -> None:
        config_path = Path(self.config_path_var.get()).expanduser()
        target_dir = config_path.parent
        target_dir.mkdir(parents=True, exist_ok=True)
        os.startfile(target_dir)


if __name__ == "__main__":
    enable_high_dpi()
    app = CodexConfigApp()
    app.mainloop()
