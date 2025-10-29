# Software-Engineering (11224120 丁誠祐 11224123 羅聖幃)
## 不寫一行程式碼，利用ChatGPT 從0 產生一個批次浮水印工具 
### 在近期的專案中，經常需要為大量圖片批次添加浮水印。傳統上這類工作通常得靠 Photoshop 手動處理，但效率極低。於是我決定結合 Python 與 ChatGPT 的力量，自動化這個流程，打造一個能快速為整個資料夾圖片加上浮水印的輕量化工具。

這個小工具的設計理念是「簡單、高效、可重複使用」。只要輸入幾個必要參數——

* 圖片資料夾路徑
* 水印字體樣式
* 水印文字內容

再點一下「確定」，程式就能自動為每張圖片生成帶有水印的版本。
不需要任何 PS 技巧，就能快速完成原本繁瑣的影像批次處理工作。

## -以下是我們利用chatgpt從0產生的程式碼-
    import os
    import tkinter as tk
    from tkinter import filedialog, messagebox
    from PIL import Image, ImageDraw, ImageFont

    class WatermarkApp:
        def __init__(self, root):
        self.root = root
        self.root.title("Watermark Generator")
        self.root.geometry("500x350")

        # === 資料綁定 ===
        self.folder_path = tk.StringVar()
        self.font_path = tk.StringVar(value="C:/Windows/Fonts/msyh.ttc")  # Windows 微軟雅黑
        self.font_size = tk.IntVar(value=20)
        self.watermark_text = tk.StringVar()

        # === 選擇圖片資料夾 ===
        tk.Label(root, text="Select Image Folder:").pack(pady=5)
        frame1 = tk.Frame(root)
        frame1.pack(pady=5)
        tk.Entry(frame1, textvariable=self.folder_path, width=45).pack(side=tk.LEFT)
        tk.Button(frame1, text="Browse", command=self.browse_folder).pack(side=tk.LEFT, padx=5)

        # === 選擇字型檔 ===
        tk.Label(root, text="Select Font:").pack(pady=5)
        frame2 = tk.Frame(root)
        frame2.pack(pady=5)
        tk.Entry(frame2, textvariable=self.font_path, width=45).pack(side=tk.LEFT)
        tk.Button(frame2, text="Browse", command=self.browse_font).pack(side=tk.LEFT, padx=5)

        # === 設定字體大小 ===
        tk.Label(root, text="Font Size:").pack(pady=5)
        tk.Entry(root, textvariable=self.font_size, width=10).pack(pady=5)

        # === 輸入浮水印文字 ===
        tk.Label(root, text="Enter Watermark Text:").pack(pady=5)
        tk.Entry(root, textvariable=self.watermark_text, width=45).pack(pady=5)

        # === 生成浮水印按鈕 ===
        tk.Button(root, text="Generate Watermark", command=self.generate_watermark).pack(pady=15)

    def browse_folder(self):
        """選擇圖片資料夾"""
        folder = filedialog.askdirectory()
        if folder:
            self.folder_path.set(folder)

    def browse_font(self):
        """選擇字型檔"""
        font_file = filedialog.askopenfilename(
            title="Select Font File",
            filetypes=[("TrueType Font", "*.ttf"), ("All files", "*.*")]
        )
        if font_file:
            self.font_path.set(font_file)

    def generate_watermark(self):
        """在圖片上生成浮水印"""
        folder = self.folder_path.get()
        text = self.watermark_text.get()
        font_file = self.font_path.get()
        font_size = self.font_size.get()

        if not folder or not text:
            messagebox.showwarning("Warning", "請選擇圖片資料夾並輸入浮水印文字！")
            return

        if not os.path.exists(font_file):
            messagebox.showwarning("Warning", "請選擇有效的字型檔 (.ttf)！")
            return

        output_folder = os.path.join(folder, "watermarked")
        os.makedirs(output_folder, exist_ok=True)

        for filename in os.listdir(folder):
            if filename.lower().endswith((".jpg", ".jpeg", ".png")):
                img_path = os.path.join(folder, filename)
                image = Image.open(img_path).convert("RGBA")

                txt_layer = Image.new("RGBA", image.size, (255, 255, 255, 0))
                draw = ImageDraw.Draw(txt_layer)
                font_style = ImageFont.truetype(font_file, font_size)

                # 計算浮水印位置（右下角）
                bbox = draw.textbbox((0, 0), text, font=font_style)
                text_width = bbox[2] - bbox[0]
                text_height = bbox[3] - bbox[1]

                x = image.width - text_width - 20
                y = image.height - text_height - 20

                # 寫上浮水印（白色 + 半透明）
                draw.text((x, y), text, font=font_style, fill=(255, 255, 255, 128))

                # 合成圖層
                watermarked = Image.alpha_composite(image, txt_layer)
                watermarked.convert("RGB").save(os.path.join(output_folder, filename))

        messagebox.showinfo("完成", f"浮水印已生成！檔案位於：\n{output_folder}")

    if __name__ == "__main__":
    root = tk.Tk()
    app = WatermarkApp(root)
    root.mainloop()
## -接下來是呈現結果-
