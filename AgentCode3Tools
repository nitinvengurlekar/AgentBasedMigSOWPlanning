import os
import io
from fpdf import FPDF
from langchain_tools import Tool  # hypothetical import for generic tool registration
from langchain_community.chat_models import ChatOpenAI
from langchain_core.messages import HumanMessage
from jinja2 import Template

# Initialize LLM
ttldb_llm = ChatOpenAI(model_name="gpt-4o", temperature=0.2)

# --- Tool 1: Generate Migration Guide & SOW ---
def generate_docs_tool(input_data: dict) -> dict:
    """
    Accepts keys: database_size, downtime_window, upgrade_required,
    current_version, target_version, target_platform, include_nonprod.
    Returns a dict with 'guide_md' and 'sow_md'.
    """
    # Fetch Oracle guide text (could be cached or stubbed here)
    guide_text = fetch_migration_guide_content(
        "https://www.oracle.com/database/cloud-migration/"
    )
    pdf_excerpt = input_data.get("pdf_excerpt")
    # Generate migration guide markdown
    prompt = (
        "You are an expert Oracle Cloud migration consultant. "
        + "Create a 3-part migration guide with hours based on inputs: \n"
        + f"DB size={input_data['database_size']}, Downtime={input_data['downtime_window']}, "
        + f"Upgrade={input_data['upgrade_required']}, Versions: {input_data['current_version']}→{input_data['target_version']}, "
        + f"Platform={input_data['target_platform']}, Include non-prod={input_data['include_nonprod']}."
    )
    migration_md = ttldb_llm([HumanMessage(content=prompt)]).content

    # Calculate total hours
    total_hours = parse_total_effort(migration_md)

    # Render SOW
    sow_md = generate_sow(
        db_size=input_data['database_size'],
        current_version=input_data['current_version'],
        target_version=input_data['target_version'],
        downtime=input_data['downtime_window'],
        platform=input_data['target_platform'],
        total_effort=total_hours,
        pdf_excerpt=pdf_excerpt or ""
    )
    return {"guide_md": migration_md, "sow_md": sow_md}

Tool1 = Tool(
    name="generate_migration_docs",
    func=generate_docs_tool,
    description="Generate migration guide and SOW based on user criteria."
)

# --- Tool 2: Convert Markdown to PDF ---
def convert_to_pdf_tool(docs: dict) -> dict:
    """
    Accepts {'guide_md': str, 'sow_md': str}. Returns PDF bytes for each.
    """
    pdf = FPDF()
    pdf.set_auto_page_break(auto=True, margin=15)

    # Guide PDF
    pdf.add_page()
    pdf.set_font("Arial", 'B', 16)
    pdf.cell(0, 10, "Oracle Migration Guide", ln=True)
    pdf.set_font("Arial", size=12)
    for line in docs['guide_md'].split("\n"):
        pdf.multi_cell(0, 8, line)

    # SOW PDF
    pdf.add_page()
    pdf.set_font("Arial", 'B', 16)
    pdf.cell(0, 10, "Statement of Work", ln=True)
    pdf.set_font("Arial", size=12)
    for line in docs['sow_md'].split("\n"):
        pdf.multi_cell(0, 8, line)

    buffer = io.BytesIO()
    pdf.output(buffer)
    buffer.seek(0)
    return {"migration_guide_pdf": buffer.read(), "sow_pdf": buffer.read()}

Tool2 = Tool(
    name="convert_to_pdf",
    func=convert_to_pdf_tool,
    description="Convert generated guide and SOW markdown into downloadable PDFs."
)

# --- Tool 3: Create Email Summary ---
def create_email_tool(email: str, docs: dict) -> dict:
    """
    Accepts an email address and docs dict. Returns email content.
    """
    subject = f"Oracle DB Migration Plan & SOW"
    body = (
        "Hello,\n\n"
        "Please find below a high-level summary of your Oracle Cloud database migration:\n\n"
        "--- Migration Guide Summary ---\n"
        + summarize_markdown(docs['guide_md'])
        + "\n\n--- Statement of Work Summary ---\n"
        + summarize_markdown(docs['sow_md'])
        + "\n\nLet me know if you have any questions.\n\nBest regards,\nYour Oracle Migration Bot"
    )
    return {"to": email, "subject": subject, "body": body}

Tool3 = Tool(
    name="create_email_summary",
    func=create_email_tool,
    description="Generate an email summary with migration guide and SOW for a given address."
)

# --- Assemble Agent ---
from langchain.agents import initialize_agent, AgentType

tools = [Tool1, Tool2, Tool3]
agent = initialize_agent(
    tools,
    ttldb_llm,
    agent_type=AgentType.OPENAI_FUNCTIONS,
    verbose=True
)

if __name__ == '__main__':
    # Example interaction
    user_query = input("How can I help you today? ")
    response = agent.run(user_query)
    print(response)
