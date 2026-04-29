import tkinter as tk
from tkinter import messagebox, ttk
import requests
import json
API_URL = "https://api.github.com/search/users"
FAV_FILE = "data.json"

def load_favorites():
    try:
        with open(FAV_FILE, 'r') as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return []

def save_favorites(favorites):
    with open(FAV_FILE, 'w') as f:
        json.dump(favorites, f, indent=4)

def search_users():
    query = entry_search.get().strip()
    
 
    if not query:
        messagebox.showwarning("Ошибка", "Поле поиска не должно быть пустым!")
        return

    try:
        response = requests.get(API_URL, params={'q': query})
        response.raise_for_status()
        data = response.json()
        
      
        for item in tree_results.get_children():
            tree_results.delete(item)
        
     
        for user in data.get('items', []):
            login = user['login']
            avatar_url = user['avatar_url']
            
           
            is_fav = any(fav['id'] == user['id'] for fav in favorites)
            btn_text = "Удалить из избранного" if is_fav else "Добавить в избранное"
            
            tree_results.insert("", tk.END, values=(login, avatar_url), tags=(btn_text,))
        
    except requests.RequestException as e:
        messagebox.showerror("Ошибка сети", f"Не удалось выполнить запрос: {e}")
    except Exception as e:
        messagebox.showerror("Ошибка", f"Произошла непредвиденная ошибка: {e}")

def toggle_favorite(event):
    item = tree_results.identify('item', event.x, event.y)
    if not item:
        return

    values = tree_results.item(item, 'values')
    login = values[0]
    

    selected_user = next((u for u in last_search_results if u['login'] == login), None)
    
    if not selected_user:
        return

    is_fav = any(fav['id'] == selected_user['id'] for fav in favorites)
    
    if is_fav:
     
        global favorites
        favorites = [fav for fav in favorites if fav['id'] != selected_user['id']]
        messagebox.showinfo("Успех", f"Пользователь {login} удален из избранного.")
    else:
        (4. Сохранение в JSON)
        favorites.append(selected_user)
        messagebox.showinfo("Успех", f"Пользователь {login} добавлен в избранное.")
    
    save_favorites(favorites)
    update_favorites_display()
    search_users() # Обновить список, чтобы изменить текст кнопок

def update_favorites_display():
    for item in tree_favorites.get_children():
        tree_favorites.delete(item)
    for user in favorites:
        tree_favorites.insert("", tk.END, values=(user['login'], user['html_url']))

favorites = load_favorites()
last_search_results = [] 

root = tk.Tk()
root.title("GitHub User Finder")
root.geometry("800x600")
root.resizable(False, False)


frame_top = tk.Frame(root)
frame_top.pack(pady=10, fill=tk.X)

entry_search = tk.Entry(frame_top, font=('Arial', 14), width=40)
entry_search.pack(side=tk.LEFT, padx=5, expand=True, fill=tk.X)

btn_search = tk.Button(frame_top, text="Найти", font=('Arial', 14), command=search_users)
btn_search.pack(side=tk.LEFT, padx=5)


frame_results = tk.LabelFrame(root, text="Результаты поиска", padx=5, pady=5)
frame_results.pack(padx=10, pady=5, fill=tk.BOTH, expand=True)

tree_results = ttk.Treeview(frame_results, columns=("Логин", "Аватар"), show="headings")
tree_results.heading("Логин", text="Логин")
tree_results.heading("Аватар", text="Аватар")
tree_results.column("Логин", width=200)
tree_results.column("Аватар", width=100)
tree_results.pack(fill=tk.BOTH, expand=True)


tree_results.bind("<Button-1>", toggle_favorite)


frame_favorites = tk.LabelFrame(root, text="Избранное", padx=5, pady=5)
frame_favorites.pack(padx=10, pady=5, fill=tk.BOTH, expand=True)

tree_favorites = ttk.Treeview(frame_favorites, columns=("Логин", "Профиль"), show="headings")
tree_favorites.heading("Логин", text="Логин")
tree_favorites.heading("Профиль", text="Ссылка на профиль")
tree_favorites.column("Логин", width=200)
tree_favorites.column("Профиль", width=400)
tree_favorites.pack(fill=tk.BOTH, expand=True)


update_favorites_display()

root.mainloop()