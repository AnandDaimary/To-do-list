import json
from datetime import datetime
from pathlib import Path

FILE_NAME = "tasks.json"


class Task:
    def __init__(self, title, priority="Medium", due_date=None, completed=False):
        self.title = title
        self.priority = priority
        self.due_date = due_date
        self.completed = completed

    def to_dict(self):
        return {
            "title": self.title,
            "priority": self.priority,
            "due_date": self.due_date,
            "completed": self.completed,
        }

    @staticmethod
    def from_dict(data):
        return Task(
            data["title"],
            data["priority"],
            data["due_date"],
            data["completed"],
        )


class TodoApp:
    def __init__(self):
        self.tasks = []
        self.load_tasks()

    def load_tasks(self):
        if Path(FILE_NAME).exists():
            with open(FILE_NAME, "r") as file:
                data = json.load(file)
                self.tasks = [Task.from_dict(task) for task in data]

    def save_tasks(self):
        with open(FILE_NAME, "w") as file:
            json.dump([task.to_dict() for task in self.tasks], file, indent=4)

    def add_task(self):
        title = input("Task title: ").strip()

        if not title:
            print("Task title cannot be empty.")
            return

        priority = input("Priority (Low/Medium/High): ").capitalize()

        if priority not in ["Low", "Medium", "High"]:
            priority = "Medium"

        due_date = input("Due date (YYYY-MM-DD) or leave blank: ").strip()

        if due_date:
            try:
                datetime.strptime(due_date, "%Y-%m-%d")
            except ValueError:
                print("Invalid date format.")
                return
        else:
            due_date = None

        task = Task(title, priority, due_date)
        self.tasks.append(task)
        self.save_tasks()

        print("Task added successfully.")

    def view_tasks(self):
        if not self.tasks:
            print("\nNo tasks available.\n")
            return

        print("\n========== TO-DO LIST ==========")

        for index, task in enumerate(self.tasks, start=1):
            status = "✓" if task.completed else "✗"

            print(f"""
{index}. [{status}] {task.title}
   Priority : {task.priority}
   Due Date: {task.due_date if task.due_date else "No due date"}
""")

    def complete_task(self):
        self.view_tasks()

        if not self.tasks:
            return

        try:
            task_number = int(input("Enter task number to mark completed: "))

            if 1 <= task_number <= len(self.tasks):
                self.tasks[task_number - 1].completed = True
                self.save_tasks()
                print("Task marked as completed.")
            else:
                print("Invalid task number.")

        except ValueError:
            print("Please enter a valid number.")

    def delete_task(self):
        self.view_tasks()

        if not self.tasks:
            return

        try:
            task_number = int(input("Enter task number to delete: "))

            if 1 <= task_number <= len(self.tasks):
                deleted = self.tasks.pop(task_number - 1)
                self.save_tasks()
                print(f"Deleted task: {deleted.title}")
            else:
                print("Invalid task number.")

        except ValueError:
            print("Please enter a valid number.")

    def menu(self):
        while True:
            print("""
==============================
      TO-DO LIST MANAGER
==============================
1. Add Task
2. View Tasks
3. Complete Task
4. Delete Task
5. Exit
""")

            choice = input("Choose an option: ")

            if choice == "1":
                self.add_task()

            elif choice == "2":
                self.view_tasks()

            elif choice == "3":
                self.complete_task()

            elif choice == "4":
                self.delete_task()

            elif choice == "5":
                print("Goodbye.")
                break

            else:
                print("Invalid choice. Try again.")


if __name__ == "__main__":
    app = TodoApp()
    app.menu()
