# mergesort-and-quick-short-visulizer
import tkinter as tk
from tkinter import ttk
import random
import time

class SortingVisualizer:
    def __init__(self, root):
        self.root = root
        self.root.title("Sorting Visualizer - Merge & Quick Sort")
        self.root.config(bg="#f0f0f0")

        # --- Control Panel ---
        control_frame = ttk.LabelFrame(root, text="Controls", padding=10)
        control_frame.pack(fill=tk.X, padx=10, pady=5)

        # Array Size
        ttk.Label(control_frame, text="Size:").grid(row=0, column=0, padx=5)
        self.size_var = tk.IntVar(value=50)
        size_spin = ttk.Spinbox(control_frame, from_=10, to=100, textvariable=self.size_var, width=5)
        size_spin.grid(row=0, column=1, padx=5)

        # Algorithm Choice
        ttk.Label(control_frame, text="Algorithm:").grid(row=0, column=2, padx=5)
        self.algo_var = tk.StringVar(value="Merge Sort")
        algo_menu = ttk.Combobox(control_frame, textvariable=self.algo_var,
                                  values=["Merge Sort", "Quick Sort"], state="readonly", width=12)
        algo_menu.grid(row=0, column=3, padx=5)

        # Speed Slider
        ttk.Label(control_frame, text="Speed:").grid(row=0, column=4, padx=5)
        self.speed_var = tk.DoubleVar(value=50)
        speed_scale = ttk.Scale(control_frame, from_=1, to=200, variable=self.speed_var, orient=tk.HORIZONTAL)
        speed_scale.grid(row=0, column=5, padx=5)
        ttk.Label(control_frame, text="Slow ←    → Fast").grid(row=0, column=6, padx=5)

        # Buttons
        ttk.Button(control_frame, text="Generate Array", command=self.generate_array).grid(row=0, column=7, padx=5)
        ttk.Button(control_frame, text="Start Sorting", command=self.start_sorting).grid(row=0, column=8, padx=5)

        # Status Label
        self.status = ttk.Label(root, text="Ready", font=(None, 10))
        self.status.pack(pady=5)

        # Canvas for bars
        self.canvas = tk.Canvas(root, width=800, height=400, bg="white")
        self.canvas.pack(padx=10, pady=5)

        # Legend
        legend_frame = ttk.Frame(root)
        legend_frame.pack(pady=5)
        for color, text in [("#808080", "Unsorted"), ("#ffd700", "Comparing"),
                            ("#ff4500", "Swapping"), ("#32cd32", "Sorted"),
                            ("#ffa500", "Left Merge"), ("#1e90ff", "Right Merge")]:
            box = tk.Canvas(legend_frame, width=15, height=15)
            box.create_rectangle(0, 0, 15, 15, fill=color)
            box.pack(side=tk.LEFT, padx=2)
            ttk.Label(legend_frame, text=text).pack(side=tk.LEFT, padx=(0,10))

        self.data = []

    def generate_array(self):
        size = self.size_var.get()
        self.data = [random.randint(10, 100) for _ in range(size)]
        self.draw_array(self.data, ["#808080"] * len(self.data))
        self.status.config(text="Array generated with {} elements".format(size))

    def draw_array(self, data, color_map):
        self.canvas.delete("all")
        c_h, c_w = 400, 800
        bar_width = c_w / len(data)

        normalized = [val / max(data) for val in data]
        for i, height in enumerate(normalized):
            x0 = i * bar_width
            y0 = c_h - height * 380
            x1 = (i + 1) * bar_width - 2
            y1 = c_h
            # Draw bar
            self.canvas.create_rectangle(x0, y0, x1, y1, fill=color_map[i])
            # Draw value on top of bar
            self.canvas.create_text((x0+x1)/2, y0 - 5, text=str(data[i]), anchor=tk.S, font=(None, 8))

        self.root.update_idletasks()

    def set_delay(self):
        speed = self.speed_var.get()
        return max(0.001, (201 - speed) / 1000)

    def start_sorting(self):
        if not self.data:
            return
        self.status.config(text=f"Sorting using {self.algo_var.get()}")
        if self.algo_var.get() == "Merge Sort":
            self.merge_sort(0, len(self.data) - 1)
        else:
            self.quick_sort(0, len(self.data) - 1)
        self.draw_array(self.data, ["#32cd32"] * len(self.data))
        self.status.config(text="Sorting Completed")

    # --------------- Merge Sort ---------------
    def merge_sort(self, left, right):
        if left < right:
            mid = (left + right) // 2
            self.merge_sort(left, mid)
            self.merge_sort(mid + 1, right)
            self.merge(left, mid, right)

    def merge(self, left, mid, right):
        left_part = self.data[left:mid + 1]
        right_part = self.data[mid + 1:right + 1]
        i = j = 0
        k = left
        colors = ["#ffa500" if left <= x <= mid else
                  "#1e90ff" if mid < x <= right else "#808080"
                  for x in range(len(self.data))]
        self.draw_array(self.data, colors)
        time.sleep(self.set_delay())

        while i < len(left_part) and j < len(right_part):
            self.status.config(text=f"Merging: {left_part[i]} and {right_part[j]}")
            if left_part[i] <= right_part[j]:
                self.data[k] = left_part[i]
                i += 1
            else:
                self.data[k] = right_part[j]
                j += 1
            k += 1
            self.draw_array(self.data, colors)
            time.sleep(self.set_delay())

        while i < len(left_part):
            self.data[k] = left_part[i]
            i += 1
            k += 1
            self.draw_array(self.data, colors)
            time.sleep(self.set_delay())

        while j < len(right_part):
            self.data[k] = right_part[j]
            j += 1
            k += 1
            self.draw_array(self.data, colors)
            time.sleep(self.set_delay())

    # --------------- Quick Sort ---------------
    def quick_sort(self, low, high):
        if low < high:
            pivot = self.partition(low, high)
            self.quick_sort(low, pivot - 1)
            self.quick_sort(pivot + 1, high)

    def partition(self, low, high):
        pivot_val = self.data[high]
        i = low - 1
        for j in range(low, high):
            self.status.config(text=f"Comparing {self.data[j]} to pivot {pivot_val}")
            self.draw_array(self.data, ["#ffd700" if x == j else
                                         "#808080" for x in range(len(self.data))])
            time.sleep(self.set_delay())
            if self.data[j] <= pivot_val:
                i += 1
                self.status.config(text=f"Swapping {self.data[i]} and {self.data[j]}")
                self.data[i], self.data[j] = self.data[j], self.data[i]
                self.draw_array(self.data, ["#ff4500" if x == i or x == j else
                                             "#808080" for x in range(len(self.data))])
                time.sleep(self.set_delay())
        self.data[i+1], self.data[high] = self.data[high], self.data[i+1]
        self.status.config(text=f"Pivot {pivot_val} to position {i+1}")
        self.draw_array(self.data, ["#32cd32" if x == i+1 else
                                     "#808080" for x in range(len(self.data))])
        time.sleep(self.set_delay())
        return i + 1

# Run the visualizer
if __name__ == '__main__':
    root = tk.Tk()
    SortingVisualizer(root)
    root.mainloop()
