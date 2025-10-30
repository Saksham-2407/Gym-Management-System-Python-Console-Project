import os
import tkinter as tk
from tkinter import ttk, messagebox

class GymMember:
    def __init__(self, member_id, name, age, gender, membership_type, contact):
        self.member_id = member_id
        self.name = name
        self.age = age
        self.gender = gender
        self.membership_type = membership_type
        self.contact = contact

    def __str__(self):
        return f"{self.member_id},{self.name},{self.age},{self.gender},{self.membership_type},{self.contact}"

class GymManagementSystem:
    def __init__(self, filename="members.txt"):
        self.filename = filename
        self.members = self.load_members()

    def load_members(self):
        members = []
        if os.path.exists(self.filename):
            with open(self.filename, "r") as file:
                for line in file:
                    data = line.strip().split(",")
                    if len(data) == 6:
                        member = GymMember(*data)
                        members.append(member)
        return members

    def save_members(self):
        with open(self.filename, "w") as file:
            for member in self.members:
                file.write(str(member) + "\n")

    def add_member(self, member_id, name, age, gender, membership_type, contact):
        if any(m.member_id == member_id for m in self.members):
            return False, "Member ID already exists."
        
        new_member = GymMember(member_id, name, age, gender, membership_type, contact)
        self.members.append(new_member)
        self.save_members()
        return True, "Member added successfully!"

    def get_member(self, member_id):
        for m in self.members:
            if m.member_id == member_id:
                return m
        return None

    def update_member(self, member_id, name, age, gender, membership_type, contact):
        for m in self.members:
            if m.member_id == member_id:
                m.name = name
                m.age = age
                m.gender = gender
                m.membership_type = membership_type
                m.contact = contact
                self.save_members()
                return True, "Member updated successfully!"
        return False, "Member not found."

    def delete_member(self, member_id):
        for m in self.members:
            if m.member_id == member_id:
                self.members.remove(m)
                self.save_members()
                return True, "Member deleted successfully!"
        return False, "Member not found."

class GymManagementGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Gym Membership Management System")
        self.root.geometry("1000x600")
        self.root.configure(bg="#2c3e50")
        
        self.system = GymManagementSystem()
        
        # Header
        header = tk.Frame(root, bg="#34495e", height=80)
        header.pack(fill=tk.X)
        
        title = tk.Label(header, text="🏋️ GYM MEMBERSHIP MANAGEMENT", 
                        font=("Arial", 24, "bold"), bg="#34495e", fg="white")
        title.pack(pady=20)
        
        # Main container
        main_container = tk.Frame(root, bg="#2c3e50")
        main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
        # Left panel - Form
        left_panel = tk.Frame(main_container, bg="#34495e", relief=tk.RAISED, bd=2)
        left_panel.pack(side=tk.LEFT, fill=tk.BOTH, padx=(0, 10), pady=0)
        
        form_title = tk.Label(left_panel, text="Member Details", 
                             font=("Arial", 16, "bold"), bg="#34495e", fg="white")
        form_title.pack(pady=15)
        
        # Form fields
        form_frame = tk.Frame(left_panel, bg="#34495e")
        form_frame.pack(padx=20, pady=10)
        
        self.entries = {}
        fields = [
            ("Member ID:", "member_id"),
            ("Name:", "name"),
            ("Age:", "age"),
            ("Gender:", "gender"),
            ("Membership Type:", "membership_type"),
            ("Contact Number:", "contact")
        ]
        
        for i, (label_text, field_name) in enumerate(fields):
            label = tk.Label(form_frame, text=label_text, bg="#34495e", 
                           fg="white", font=("Arial", 11))
            label.grid(row=i, column=0, sticky=tk.W, pady=8)
            
            if field_name == "gender":
                entry = ttk.Combobox(form_frame, width=25, 
                                    values=["M", "F", "Other"], state="readonly")
                entry.current(0)
            elif field_name == "membership_type":
                entry = ttk.Combobox(form_frame, width=25,
                                    values=["Monthly", "Quarterly", "Yearly"], 
                                    state="readonly")
                entry.current(0)
            else:
                entry = tk.Entry(form_frame, width=27, font=("Arial", 10))
            
            entry.grid(row=i, column=1, pady=8, padx=10)
            self.entries[field_name] = entry
        
        # Buttons
        button_frame = tk.Frame(left_panel, bg="#34495e")
        button_frame.pack(pady=20)
        
        btn_style = {"width": 12, "height": 2, "font": ("Arial", 10, "bold"), 
                    "cursor": "hand2", "relief": tk.RAISED, "bd": 2}
        
        add_btn = tk.Button(button_frame, text="Add", bg="#27ae60", fg="white",
                           command=self.add_member, **btn_style)
        add_btn.grid(row=0, column=0, padx=5, pady=5)
        
        update_btn = tk.Button(button_frame, text="Update", bg="#f39c12", fg="white",
                              command=self.update_member, **btn_style)
        update_btn.grid(row=0, column=1, padx=5, pady=5)
        
        delete_btn = tk.Button(button_frame, text="Delete", bg="#e74c3c", fg="white",
                              command=self.delete_member, **btn_style)
        delete_btn.grid(row=1, column=0, padx=5, pady=5)
        
        clear_btn = tk.Button(button_frame, text="Clear", bg="#95a5a6", fg="white",
                             command=self.clear_fields, **btn_style)
        clear_btn.grid(row=1, column=1, padx=5, pady=5)
        
        # Right panel - Table
        right_panel = tk.Frame(main_container, bg="#34495e", relief=tk.RAISED, bd=2)
        right_panel.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
        
        table_title = tk.Label(right_panel, text="Members List", 
                              font=("Arial", 16, "bold"), bg="#34495e", fg="white")
        table_title.pack(pady=15)
        
        # Search bar
        search_frame = tk.Frame(right_panel, bg="#34495e")
        search_frame.pack(pady=10)
        
        tk.Label(search_frame, text="Search by ID:", bg="#34495e", 
                fg="white", font=("Arial", 11)).pack(side=tk.LEFT, padx=5)
        
        self.search_entry = tk.Entry(search_frame, width=20, font=("Arial", 10))
        self.search_entry.pack(side=tk.LEFT, padx=5)
        
        search_btn = tk.Button(search_frame, text="Search", bg="#3498db", fg="white",
                              command=self.search_member, width=10, height=1,
                              font=("Arial", 10, "bold"), cursor="hand2")
        search_btn.pack(side=tk.LEFT, padx=5)
        
        refresh_btn = tk.Button(search_frame, text="Refresh", bg="#16a085", fg="white",
                               command=self.refresh_table, width=10, height=1,
                               font=("Arial", 10, "bold"), cursor="hand2")
        refresh_btn.pack(side=tk.LEFT, padx=5)
        
        # Treeview
        tree_frame = tk.Frame(right_panel, bg="#34495e")
        tree_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        scrollbar = ttk.Scrollbar(tree_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        columns = ("ID", "Name", "Age", "Gender", "Type", "Contact")
        self.tree = ttk.Treeview(tree_frame, columns=columns, show="headings",
                                yscrollcommand=scrollbar.set, height=15)
        
        scrollbar.config(command=self.tree.yview)
        
        # Column headings
        col_widths = [80, 120, 60, 80, 100, 120]
        for col, width in zip(columns, col_widths):
            self.tree.heading(col, text=col)
            self.tree.column(col, width=width, anchor=tk.CENTER)
        
        self.tree.pack(fill=tk.BOTH, expand=True)
        self.tree.bind("<ButtonRelease-1>", self.on_tree_select)
        
        # Load initial data
        self.refresh_table()
    
    def clear_fields(self):
        for field_name, entry in self.entries.items():
            if isinstance(entry, ttk.Combobox):
                entry.current(0)
            else:
                entry.delete(0, tk.END)
    
    def refresh_table(self):
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        for member in self.system.members:
            self.tree.insert("", tk.END, values=(
                member.member_id, member.name, member.age,
                member.gender, member.membership_type, member.contact
            ))
    
    def add_member(self):
        values = {k: v.get() for k, v in self.entries.items()}
        
        if not all(values.values()):
            messagebox.showwarning("Warning", "All fields are required!")
            return
        
        success, message = self.system.add_member(**values)
        
        if success:
            messagebox.showinfo("Success", message)
            self.clear_fields()
            self.refresh_table()
        else:
            messagebox.showerror("Error", message)
    
    def update_member(self):
        values = {k: v.get() for k, v in self.entries.items()}
        
        if not all(values.values()):
            messagebox.showwarning("Warning", "All fields are required!")
            return
        
        success, message = self.system.update_member(**values)
        
        if success:
            messagebox.showinfo("Success", message)
            self.clear_fields()
            self.refresh_table()
        else:
            messagebox.showerror("Error", message)
    
    def delete_member(self):
        member_id = self.entries["member_id"].get()
        
        if not member_id:
            messagebox.showwarning("Warning", "Please enter Member ID!")
            return
        
        confirm = messagebox.askyesno("Confirm", 
                                     f"Delete member {member_id}?")
        if confirm:
            success, message = self.system.delete_member(member_id)
            
            if success:
                messagebox.showinfo("Success", message)
                self.clear_fields()
                self.refresh_table()
            else:
                messagebox.showerror("Error", message)
    
    def search_member(self):
        member_id = self.search_entry.get()
        
        if not member_id:
            messagebox.showwarning("Warning", "Please enter Member ID to search!")
            return
        
        member = self.system.get_member(member_id)
        
        if member:
            for item in self.tree.get_children():
                self.tree.delete(item)
            
            self.tree.insert("", tk.END, values=(
                member.member_id, member.name, member.age,
                member.gender, member.membership_type, member.contact
            ))
        else:
            messagebox.showinfo("Not Found", "Member not found!")
    
    def on_tree_select(self, event):
        selected = self.tree.selection()
        if selected:
            item = self.tree.item(selected[0])
            values = item["values"]
            
            self.entries["member_id"].delete(0, tk.END)
            self.entries["member_id"].insert(0, values[0])
            
            self.entries["name"].delete(0, tk.END)
            self.entries["name"].insert(0, values[1])
            
            self.entries["age"].delete(0, tk.END)
            self.entries["age"].insert(0, values[2])
            
            self.entries["gender"].set(values[3])
            self.entries["membership_type"].set(values[4])
            
            self.entries["contact"].delete(0, tk.END)
            self.entries["contact"].insert(0, values[5])

def main():
    root = tk.Tk()
    app = GymManagementGUI(root)
    root.mainloop()

if __name__ == "__main__":
    main()
