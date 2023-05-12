# Easylist-pfrom tkinter import *
from tkinter import messagebox
import sqlite3
root = Tk()
# Create PhotoImage objects for images with alternate text
image1 = PhotoImage(file="C:/Users/Abimb/OneDrive/Desktop/Easylist-logo.png")
image2 = PhotoImage(file="C:/Users/Abimb/OneDrive/Desktop/EasyList2.png")
# Create Label widgets for images with alternate text
image1_label = Label(root, image=image1, bd=0)
image1_label.image = image1
image1_label.pack(pady=5)
image2_label = Label(root, image=image2, bd=0)
image2_label.image = image2
image2_label.pack(pady=5)
root.mainloop()

# declare the global variables
global list_name, priority, items_listbox, current_list_id

# create the main window
root = Tk()
root.title("EasyList")

# connect to the database
conn = sqlite3.connect('easylist.db')
c = conn.cursor()

# create the lists table if it doesn't exist
c.execute('''CREATE TABLE IF NOT EXISTS lists
             (id INTEGER PRIMARY KEY AUTOINCREMENT,
              name TEXT,
              priority TEXT)''')

# create the items table if it doesn't exist
c.execute('''CREATE TABLE IF NOT EXISTS items
             (id INTEGER PRIMARY KEY AUTOINCREMENT,
              list_id INTEGER,
              name TEXT,
              finished INTEGER,
              priority INTEGER,
              FOREIGN KEY(list_id) REFERENCES lists(id))''')

# function to create a new list


def new_list():
    global list_name_entry, priority_entry, new_list_window

    # create a new window for the new list
    new_list_window = Toplevel(root)
    new_list_window.title("New List")

    # create and position the list name label and entry box
    list_name_label = Label(new_list_window, text="List Name:")
    list_name_label.grid(row=0, column=0, padx=5, pady=5)

    list_name_entry = Entry(new_list_window)
    list_name_entry.grid(row=0, column=1, padx=5, pady=5)

    # create and position the priority label and entry box
    priority_label = Label(new_list_window, text="Priority Level (Optional):")
    priority_label.grid(row=1, column=0, padx=5, pady=5)

    priority_entry = Entry(new_list_window)
    priority_entry.grid(row=1, column=1, padx=5, pady=5)

    # create and position the save button
    save_button = Button(new_list_window, text="Save", command=save_list)
    save_button.grid(row=2, column=0, columnspan=2, padx=5, pady=5)

# function to save the new list and close the new list window


def save_list():
    global list_name_entry, priority_entry, list_name, priority, items_listbox, new_list_window, current_list_id

    # get the values from the entry boxes
    list_name = list_name_entry.get()
    priority = priority_entry.get()

    # validate the input
    if not list_name:
        messagebox.showerror("Error", "List name cannot be empty.")
        return

    # close the new list window
    new_list_window.destroy()

    # insert the list into the database
    c.execute("INSERT INTO lists (name, priority) VALUES (?, ?)",
              (list_name, priority))
    conn.commit()

    # get the id of the new list
    current_list_id = c.lastrowid

    # create a new window for the list
    list_window = Toplevel(root)
    list_window.title(list_name)

    # create and position the list name label
    list_name_label = Label(list_window, text=list_name)
    list_name_label.grid(row=0, column=0, padx=5, pady=5)

    # create and position the add item button and entry box
    add_item_entry = Entry(list_window)
    add_item_entry.grid(row=1, column=0, padx=5, pady=5)

    def add_item():
        global items_listbox

        # get the value from the entry box
        item = add_item_entry.get()

        # validate the input
        if not item:
            messagebox.showerror("Error", "Item cannot be empty.")
            return

        # add the item to the database
        c.execute("INSERT INTO items (list_id, name, finished, priority) VALUES (?, ?, ?, ?)",
                  (current_list_id, item, 0, 0))
        conn.commit()

        # add the item to the listbox
        items_listbox.insert(END, item)

        # clear the entry box
        add_item_entry.delete(0, END)

    add_item_button = Button(list_window, text="Add Item", command=add_item)
    add_item_button.grid(row=1, column=1, padx=5, pady=5)

    # create and position the items listbox
    items_listbox = Listbox(list_window)
    items_listbox.grid(row=2, column=0, columnspan=2, padx=5, pady=5)

    # get the items from the database
    c.execute(
        "SELECT name, finished, priority FROM items WHERE list_id=?", (current_list_id,))
    items = c.fetchall()

    # add the items to the listbox
    for item in items:
        name = item[0]
        finished = item[1]
        priority = item[2]
        if finished:
            items_listbox.insert(END, name + " (finished)")
        else:
            items_listbox.insert(END, name)

    # create and position the remove item button
    def remove_item():
        global items_listbox

        # get the selected item
        selected_item = items_listbox.curselection()
        if not selected_item:
            messagebox.showerror("Error", "Please select an item to remove.")
            return
        item = items_listbox.get(selected_item[0])

        # remove the item from the database
        c.execute("DELETE FROM items WHERE list_id=? AND name=?",
                  (current_list_id, item))
        conn.commit()

        # remove the item from the listbox
        items_listbox.delete(selected_item[0])

    remove_item_button = Button(
        list_window, text="Remove Item", command=remove_item)
    remove_item_button.grid(row=3, column=0, padx=5, pady=5)

    # create and position the mark finished button
    def mark_finished():
        global items_listbox

        # get the selected item
        selected_item = items_listbox.curselection()
        if not selected_item:
            messagebox.showerror(
                "Error", "Please select an item to mark as finished.")
            return
        item = items_listbox.get(selected_item[0])

        # mark the item as finished in the database
        c.execute("UPDATE items SET finished=1 WHERE list_id=? AND name=?",
                  (current_list_id, item))
        conn.commit()

        # add the "finished" tag to the item in the listbox
        items_listbox.delete(selected_item[0])
        items_listbox.insert(selected_item[0], item + " (finished)")

    mark_finished_button = Button(
        list_window, text="Mark Finished", command=mark_finished)
    mark_finished_button.grid(row=3, column=1, padx=5, pady=5)

    # create and position the mark unfinished button
    def mark_unfinished():
        mark_unfinished_button = Button(
            list_window, text="Mark Unfinished", command=mark_unfinished)
        mark_unfinished_button.grid(row=4, column=0, padx=5, pady=5)

    # create and position the sort by name button
    def sort_by_name():
        global items_listbox

        # get the items from the database, sorted by name
        c.execute(
            "SELECT name, finished, priority FROM items WHERE list_id=? ORDER BY name", (current_list_id,))
        items = c.fetchall()

        # clear the listbox
        items_listbox.delete(0, END)

        # add the items to the listbox
        for item in items:
            name = item[0]
            finished = item[1]
            priority = item[2]
            if finished:
                items_listbox.insert(END, name + " (finished)")
            else:
                items_listbox.insert(END, name)

    sort_by_name_button = Button(
        list_window, text="Sort by Name", command=sort_by_name)
    sort_by_name_button.grid(row=4, column=1, padx=5, pady=5)

    # create and position the back to menu button

    def back_to_menu():
        list_window.destroy()

    back_to_menu_button = Button(
        list_window, text="Back to Menu", command=back_to_menu)
    back_to_menu_button.grid(row=6, column=0, columnspan=2, padx=5, pady=5)


# function to show the items in the selected list
def show_items(list_id):
    global items_listbox

    # create a new window for the list items
    list_window = Toplevel(root)
    list_window.title("List Items")

    # # create and position the listbox
    # items_listbox = Listbox(list_window)
    # items_listbox.grid(row=1, column=0, padx=5, pady=5)

    # get the items from the database for the selected list
    c.execute("SELECT name, finished FROM items WHERE list_id=?", (list_id,))
    items = c.fetchall()

    # # add the items to the listbox
    # for item in items:
    #     name = item[0]
    #     finished = item[1]
    #     items_listbox.insert(END, name + (" (Finished)" if finished else ""))

    add_item_entry = Entry(list_window)
    add_item_entry.grid(row=1, column=0, padx=5, pady=5)
    # create and position the add item button

    def add_item():
        global items_listbox

        # get the value from the entry box
        item = add_item_entry.get()

        # validate the input
        if not item:
            messagebox.showerror("Error", "Item cannot be empty.")
            return

        # add the item to the database
        c.execute("INSERT INTO items (list_id, name, finished, priority) VALUES (?, ?, ?, ?)",
                  (current_list_id, item, 0, 0))
        conn.commit()

        # add the item to the listbox
        items_listbox.insert(END, item)

        # clear the entry box
        add_item_entry.delete(0, END)

    add_item_button = Button(list_window, text="Add Item", command=add_item)
    add_item_button.grid(row=1, column=1, padx=5, pady=5)

    # create and position the items listbox
    items_listbox = Listbox(list_window)
    items_listbox.grid(row=2, column=0, columnspan=2, padx=5, pady=5)

    # get the items from the database
    c.execute(
        "SELECT name, finished, priority FROM items WHERE list_id=?", (current_list_id,))
    items = c.fetchall()

    # add the items to the listbox
    for item in items:
        name = item[0]
        finished = item[1]
        if finished:
            items_listbox.insert(END, name + " (finished)")
        else:
            items_listbox.insert(END, name)

    # create and position the remove item button
    def remove_item():
        global items_listbox

        # get the selected item
        selected_item = items_listbox.curselection()
        if not selected_item:
            messagebox.showerror("Error", "Please select an item to remove.")
            return
        item = items_listbox.get(selected_item[0])

        # remove the item from the database
        c.execute("DELETE FROM items WHERE list_id=? AND name=?",
                  (current_list_id, item))
        conn.commit()

        # remove the item from the listbox
        items_listbox.delete(selected_item[0])

    remove_item_button = Button(
        list_window, text="Remove Item", command=remove_item)
    remove_item_button.grid(row=3, column=0, padx=5, pady=5)

    # create and position the mark finished button
    def mark_finished():
        global items_listbox

        # get the selected item
        selected_item = items_listbox.curselection()
        if not selected_item:
            messagebox.showerror(
                "Error", "Please select an item to mark as finished.")
            return
        item = items_listbox.get(selected_item[0])

        # mark the item as finished in the database
        c.execute("UPDATE items SET finished=1 WHERE list_id=? AND name=?",
                  (current_list_id, item))
        conn.commit()

        # add the "finished" tag to the item in the listbox
        items_listbox.delete(selected_item[0])
        items_listbox.insert(selected_item[0], item + " (finished)")

    mark_finished_button = Button(
        list_window, text="Mark Finished", command=mark_finished)
    mark_finished_button.grid(row=3, column=1, padx=5, pady=5)

    # create and position the mark unfinished button
    def mark_unfinished():
        global items_listbox

        # get the selected item
        selected_item = items_listbox.curselection()
        if not selected_item:
            messagebox.showerror(
                "Error", "Please select an item to mark as unfinished.")
            return
        item = items_listbox.get(selected_item[0])

        # mark the item as finished in the database
        c.execute("UPDATE items SET finished=1 WHERE list_id=? AND name=?",
                  (current_list_id, item))
        conn.commit()

        # add the "finished" tag to the item in the listbox
        items_listbox.delete(selected_item[0])
        items_listbox.insert(selected_item[0], item)

    mark_unfinished_button = Button(
        list_window, text="Mark Unfinished", command=mark_unfinished)
    mark_unfinished_button.grid(row=4, column=0, padx=5, pady=5)

    # create and position the sort by name button
    def sort_by_name():
        global items_listbox

        # get the items from the database, sorted by name
        c.execute(
            "SELECT name, finished, priority FROM items WHERE list_id=? ORDER BY name", (current_list_id,))
        items = c.fetchall()

        # clear the listbox
        items_listbox.delete(0, END)

        # add the items to the listbox
        for item in items:
            name = item[0]
            finished = item[1]
            if finished:
                items_listbox.insert(END, name + " (finished)")
            else:
                items_listbox.insert(END, name)

    sort_by_name_button = Button(
        list_window, text="Sort by Name", command=sort_by_name)
    sort_by_name_button.grid(row=4, column=1, padx=5, pady=5)

    # create and position the back to menu button

    def back_to_menu():
        list_window.destroy()

    back_to_menu_button = Button(
        list_window, text="Back to Menu", command=back_to_menu)
    back_to_menu_button.grid(row=6, column=0, columnspan=2, padx=5, pady=5)


# function to show the user's lists
def show_lists():
    global list_select_listbox

    # create a new window for the list selection
    lists_window = Toplevel(root)
    lists_window.title("Lists")

    # create and position the label
    list_select_label = Label(lists_window, text="Select a List:")
    list_select_label.grid(row=0, column=0, padx=5, pady=5)

    # create and position the listbox
    list_select_listbox = Listbox(lists_window)
    list_select_listbox.grid(row=1, column=0, padx=5, pady=5)

    # get the lists from the database
    c.execute("SELECT name FROM lists")
    lists = c.fetchall()

    # add the lists to the listbox
    for lst in lists:
        list_select_listbox.insert(END, lst[0])

    # create and position the open list button
    def open_list():
        global current_list_id

        # get the selected list
        selected_list = list_select_listbox.curselection()
        if not selected_list:
            messagebox.showerror("Error", "Please select a list to open.")
            return
        list_name = list_select_listbox.get(selected_list[0])

        # get the list id from the database
        c.execute("SELECT id FROM lists WHERE name=?", (list_name,))
        row = c.fetchone()
        if not row:
            messagebox.showerror("Error", "List not found.")
            return
        current_list_id = row[0]

        # destroy the lists window
        lists_window.destroy()

        # show the items in the selected list
        show_items(current_list_id)

    open_list_button = Button(
        lists_window, text="Open List", command=open_list)
    open_list_button.grid(row=2, column=0, padx=5, pady=5)

    # create and position the new list button
    def new_list():
        # create a new window for the new list
        new_list_window = Toplevel(lists_window)
        new_list_window.title("New List")

        # create and position the name label and entry box
        new_list_label = Label(new_list_window, text="Name:")
        new_list_label.grid(row=0, column=0, padx=5, pady=5)
        new_list_entry = Entry(new_list_window)
        new_list_entry.grid(row=0, column=1, padx=5, pady=5)

        # create and position the priority label and entry box
        new_list_priority_label = Label(new_list_window, text="Priority:")
        new_list_priority_label.grid(row=1, column=0, padx=5, pady=5)
        new_list_priority_entry = Entry(new_list_window)
        new_list_priority_entry.grid(row=1, column=1, padx=5, pady=5)

        # create and position the create button
        def create_list():
            # get the values from the entry boxes
            name = new_list_entry.get()
            priority = new_list_priority_entry.get()

            # validate the input
            if not name:
                messagebox.showerror("Error", "Name cannot be empty.")
                return

            # add the list to the database
            c.execute(
                "INSERT INTO lists (name, priority) VALUES (?, ?)", (name, priority))
            conn.commit()

            # destroy the new list window
            new_list_window.destroy()

            # update the listbox
            list_select_listbox.insert(END, name)
            if priority:
                list_select_listbox.insert(
                    END, f"{name} (priority {priority})")
            else:
                list_select_listbox.insert(END, name)

        create_list_button = Button(
            new_list_window, text="Create", command=create_list)
        create_list_button.grid(row=2, column=0, columnspan=2, padx=5, pady=5)

    new_list_button = Button(lists_window, text="New List", command=new_list)
    new_list_button.grid(row=3, column=0, padx=5, pady=5)


# create and position the main menu buttons
new_list_button = Button(root, text="New List", command=new_list)
new_list_button.grid(row=0, column=0, padx=5, pady=5)

show_lists_button = Button(root, text="Show Lists", command=show_lists)
show_lists_button.grid(row=0, column=1, padx=5, pady=5)

# create and position the exit button


def exit_program():
    root.destroy()


exit_button = Button(root, text="Exit", command=exit_program)
exit_button.grid(row=1, column=0, padx=5, pady=5)

root.mainloop()
rogramming
