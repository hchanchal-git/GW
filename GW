from jira import JIRA
import base64
import os
import webbrowser
import shutil
import subprocess
import platform
# Jira credentials and server
jira_server = 'https://beetle.egain.com/'
jira_user = 'hchanchal'  # Please provide your JIRA userID
# ============================
# Decode JIRA password from environment
# ============================
try:
    encoded_jira_pass = os.environ['JIRA_PASS']
    decoded_bytes = base64.b64decode(encoded_jira_pass)
    jira_pass = decoded_bytes.decode('utf-8')
except KeyError:
    print("❌ Error: JIRA_PASS environment variable not set. Please set the JIRA_PASS environment variable.")
    exit()
# ============================
# Connect to Jira
# ============================
try:
    jira = JIRA(server=jira_server, basic_auth=(jira_user, jira_pass))
except Exception as e:
    print(f"❌ Error connecting to Jira: {e}")
    exit()
# ============================
# Custom field IDs
# ============================
SYMPTOM_FIELD = "customfield_10020"
REPRO_STEPS_FIELD = "customfield_10019"
PRELIM_ANALYSIS_FIELD = "customfield_15540"
ACTIONABLE_BY_FIELD = "customfield_10930"
CASE_PRIORITY_FIELD = "customfield_14936"
REASON_REPORTED_FIELD = "customfield_18031"
REPORTED_BY_FIELD = "customfield_18835"
AFFECTS_HOTFIX_FIELD = "customfield_12134"
IS_R21_UPGRADE_FIELD = "customfield_18830"
INSTANCE_NAMES_FIELD = "customfield_17932"
ORIGINAL_ISSUE_KEY = "CBU-258862"  # Template for Gather Logs
# ============================
# Browser opening logic
# ============================
def open_link_in_browser(url):
    browsers = ['edge', 'chrome', 'firefox']
    browser_execs = {
        'edge': {
            'Windows': 'msedge',
            'Linux': 'microsoft-edge',
            'Darwin': 'microsoft edge'
        },
        'chrome': {
            'Windows': 'chrome',
            'Linux': 'google-chrome',
            'Darwin': 'google chrome'
        },
        'firefox': {
            'Windows': 'firefox',
            'Linux': 'firefox',
            'Darwin': 'firefox'
        }
    }
    os_name = platform.system()
    for browser in browsers:
        exec_name = browser_execs[browser].get(os_name)
        if not exec_name:
            continue
        if shutil.which(exec_name):
            try:
                browser_obj = webbrowser.get(using=exec_name)
                browser_obj.open_new(url)
                print(f"🌐 Opened in {browser.capitalize()}")
                return
            except webbrowser.Error:
                continue
    print("⚠️ Could not find Edge, Chrome, or Firefox. Opening with default browser.")
    webbrowser.open_new(url)
# ============================
# Main logic
# ============================
def main():
    try:
        print("✅ Successfully connected to Jira.")
        server_name = input("Enter server name: ").strip().upper()
        alert_type = input("Enter Type_Of_Alert: ").strip()
        if not server_name or not alert_type:
            print("❌ Server name and Alert Type cannot be empty. Aborting.")
            return
        # === Step 1: Create Action Item ===
        print("\n--- Step 1: Creating Action Item ---")
        summary_text = f"Investigate {alert_type} generated for {server_name}"
        symptom_text = f"Alert generated. Investigate {alert_type} generated for {server_name}"
        action_item_fields = {
            "project": {"key": "ACTION"},
            "issuetype": {"name": "Action Item"},
            "summary": summary_text,
            "description": summary_text,
            "labels": ["LOGS_MONITORING"],
            "components": [{"name": "Internal Issue"}],
            SYMPTOM_FIELD: symptom_text,
            REPRO_STEPS_FIELD: "N/A",
            PRELIM_ANALYSIS_FIELD: "N/A",
            ACTIONABLE_BY_FIELD: {"value": "Product Engineering"},
            CASE_PRIORITY_FIELD: {"value": "P2"},
            REASON_REPORTED_FIELD: {"value": "Internal Request"},
            REPORTED_BY_FIELD: {"value": "Cloud Ops"},
            AFFECTS_HOTFIX_FIELD: "21.21.0",
            IS_R21_UPGRADE_FIELD: {"value": "No"},
            "assignee": {"name": "jchoudhary"},
        }
        action_issue = jira.create_issue(fields=action_item_fields)
        action_url = f"{jira_server}browse/{action_issue.key}"
        print(f"✅ Action Item created: {action_issue.key}")
        print(f"🔗 {action_url}")
        open_link_in_browser(action_url)
        # === Step 2: Create Gather Logs issue ===
        print("\n--- Step 2: Creating Gather Logs Issue ---")
        template_issue = jira.issue(ORIGINAL_ISSUE_KEY)
        desc = template_issue.fields.description if template_issue.fields.description else template_issue.fields.summary
        desc = str(desc).replace("<server_name>", server_name)
        gather_logs_fields = {
            "project": {"key": template_issue.fields.project.key},
            "issuetype": {"id": template_issue.fields.issuetype.id},
            "summary": f"Gather logs for {server_name}",
            INSTANCE_NAMES_FIELD: server_name,
            "description": desc,
        }
        if getattr(template_issue.fields.issuetype, "subtask", False) and hasattr(template_issue.fields, 'parent'):
            gather_logs_fields["parent"] = {"key": template_issue.fields.parent.key}
        logs_issue = jira.create_issue(fields=gather_logs_fields)
        jira.assign_issue(logs_issue, "cloudbot")
        logs_url = f"{jira_server}browse/{logs_issue.key}"
        print(f"✅ Gather Logs issue created and assigned to cloudbot: {logs_issue.key}")
        print(f"🔗 {logs_url}")
        open_link_in_browser(logs_url)
        # === Step 3: Link both issues ===
        print("\n--- Step 3: Linking Issues ---")
        LINK_TYPE = "is related to"
        jira.create_issue_link(type=LINK_TYPE, inwardIssue=action_issue.key, outwardIssue=logs_issue.key)
        print(f"✅ Issues linked successfully as '{LINK_TYPE}' between {action_issue.key} and {logs_issue.key}")
    except Exception as e:
        print(f"\n❌ Script execution failed. Check the error message for details:")
        print(e)
# ============================
# Entry point
# ============================
if __name__ == "__main__":
    main()
