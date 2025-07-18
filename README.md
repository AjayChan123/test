# test
demo repo

# New Section
I added this section to demonstrate committing a change to a Read Me file in Github.

# FocusFlow: A Smart Task Timer and Personal Discipline Coach
# Author: Ajay Chanthavisouk

from flask import Flask, render_template_string, request, redirect, url_for
import threading
import time

app = Flask(__name__)

# Default timer length (25 minutes for focus, 5 for break)
DEFAULT_FOCUS = 25 * 60
DEFAULT_BREAK = 5 * 60

# Store session info
session_data = {
    "mode": "focus",          # focus or break
    "time_left": DEFAULT_FOCUS,
    "running": False,
    "task_name": "",
    "log": []
}

# HTML template
TEMPLATE = """
<!doctype html>
<html>
    <head>
        <title>FocusFlow</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
            h1 { color: #2c3e50; }
            .timer { font-size: 48px; margin: 20px 0; }
            .mode { font-size: 24px; color: #16a085; }
        </style>
    </head>
    <body>
        <h1>FocusFlow</h1>
        <p class="mode">Mode: {{ mode.title() }}</p>
        <p class="timer">{{ time_left }}</p>

        <form method="POST">
            <input type="text" name="task" placeholder="Enter task name" value="{{ task_name }}">
            <br><br>
            <button name="action" value="start">Start</button>
            <button name="action" value="pause">Pause</button>
            <button name="action" value="reset">Reset</button>
        </form>

        <h3>Session Log</h3>
        <ul>
            {% for entry in log %}
                <li>{{ entry }}</li>
            {% endfor %}
        </ul>
    </body>
</html>
"""

# Timer thread
def run_timer():
    while session_data["running"] and session_data["time_left"] > 0:
        time.sleep(1)
        session_data["time_left"] -= 1

    if session_data["time_left"] == 0:
        session_data["running"] = False
        mode = session_data["mode"]
        task = session_data["task_name"] or "Unnamed Task"
        log_entry = f"Completed {mode} session for task: {task}"
        session_data["log"].append(log_entry)

        # Auto-switch mode
        if mode == "focus":
            session_data["mode"] = "break"
            session_data["time_left"] = DEFAULT_BREAK
        else:
            session_data["mode"] = "focus"
            session_data["time_left"] = DEFAULT_FOCUS

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        action = request.form.get("action")
        task = request.form.get("task", "").strip()
        if task:
            session_data["task_name"] = task

        if action == "start" and not session_data["running"]:
            session_data["running"] = True
            threading.Thread(target=run_timer).start()
        elif action == "pause":
            session_data["running"] = False
        elif action == "reset":
            session_data["running"] = False
            session_data["time_left"] = DEFAULT_FOCUS if session_data["mode"] == "focus" else DEFAULT_BREAK
            session_data["task_name"] = ""

    minutes = session_data["time_left"] // 60
    seconds = session_data["time_left"] % 60
    formatted_time = f"{minutes:02}:{seconds:02}"

    return render_template_string(
        TEMPLATE,
        time_left=formatted_time,
        mode=session_data["mode"],
        task_name=session_data["task_name"],
        log=session_data["log"]
    )

if __name__ == "__main__":
    app.run(debug=True)
#1stcode 
