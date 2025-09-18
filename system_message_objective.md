# Detailed Deep Analysis of System Messages for LLM Trainer Tasks

# System Message should be minimum of 11K character in length.

This document synthesizes the findings from the individual task analyses to provide a set of "liquid gold" insights and actionable tips for LLM trainers. The primary goal is to move beyond simple analysis and to understand the art of constructing system messages (SMs) that test the boundaries of a model's capabilities.

The core philosophy is that for training a robust, tool-using agent, success is good, but strategic failure is better. A well-designed SM should create complex scenarios that push the model to the edge of its capacity, causing it to fail in predictable ways. These failures, when corrected, provide invaluable training data, teaching the model to handle nuance, ambiguity, and complex multi-step automation.

## Key Principles of Effective System Message Construction (The "Edge Case" Framework)

Based on the analysis, several key principles are critical for constructing SMs that effectively test and train a model.

### 1. Start with a Strong Foundation to Define the Playing Field

Before you can test the model's limits, you must clearly define the boundaries of the task. The most effective SMs establish a strong foundation of context.

- **Bot Persona**: Defines the bot's expected behavior, tone, and expertise.
- **User Profile**: Informs the model about the user's knowledge level, goals, and likely communication style.
- **Operational Context**: Anchors the conversation in a specific state, providing the model with the immediate information it needs to act.

**Creating the Edge Case**: A well-defined persona can be used to create complex behavioral constraints. For example, a "cautious" persona should be more likely to ask for confirmation, while a "proactive" persona should suggest next steps.

**Example (from Task 807235: OmniRetail Assistant)**:
- **SM Snippet**: "You are Alex, the OmniRetail Assistant, an advanced AI designed to streamline and optimize e-commerce operations... You are meticulous, proactive, and focused on operational excellence."
- **User Prompt**: "How many have ordered this Hair Dryer here recently?"
- **The Test**: The word "recently" is deliberately vague. A "meticulous" and "proactive" assistant should not guess the time frame.
- **Model Failure (and subsequent correction)**: The model initially assumed "last month." The trainer corrected this by having the model ask for clarification: "When you say 'recently,' could you specify the time period...?" This trains the model to handle ambiguity in line with its persona.

### 2. Structure for Clarity and to Test Logical Reasoning

A well-structured SM makes it easier for the model to understand the instructions, but it can also be used to test the model's ability to reason and prioritize.

- **General Principles First**: High-level rules that should always be followed.
- **Workflow-Centric Organization**: Grouping instructions by task (e.g., "Listing Management," "Order Management").
- **Tool-Specific Sections**: Providing detailed instructions for complex tools.

**Creating the Edge Case**: You can test the model's ability to prioritize by providing conflicting or overlapping rules in different sections. The model must then decide which rule takes precedence.

**Example (from Task 807225: Inventory Management Assistant)**:
- **SM Snippet**: "All the report fetches must be driven sequentially by prioritizing the report statuses; When the user requests some reports, the driving factor should never be the report types; instead, fetch all the reports in the DONE state, then the priority goes like this: In-progress -> In-Queue -> Cancelled."
- **User Prompt**: "Fetch me their latest sales report to review."
- **The Test**: This prompt is designed to see if the model will correctly prioritize fetching DONE reports over simply searching for "sales" reports.
- **Model Behavior**: The model correctly calls get_reports with processing_statuses: ["DONE"], demonstrating its ability to follow the prioritization rule.

### 3. Provide Clear Instructions with Deliberate Ambiguity

The core of the SM is the set of instructions. To create good training data, these instructions should be a mix of crystal-clear rules and deliberately ambiguous scenarios.

**Creating the Edge Case**: Introduce scenarios where the user's intent is not immediately obvious. This forces the model to use its reasoning abilities to infer the correct course of action, or to ask for clarification.

**Example (from Task 807238: Amazon Product Launch Assistant)**:
- **SM Snippet**: "If the user asks to check whether they are in a channel, first check in the private channels because the user is an Amazon Seller and Marketing Manager. If no relevant channel is found in the private channels, include both private and public channels in the next tool call."
- **User Prompt**: "Am I connected to the marketing channel in Slack?"
- **The Test**: This tests the model's ability to follow a two-step logic: first, check private channels, then, if necessary, check all channels.
- **Model Behavior**: A well-trained model would first call conversations_list with types: "private_channel". If that fails, it would then call it again with types: "public_channel,private_channel".

### 4. Guide, Don't Just Command, to Test Proactivity

The best assistants don't just follow orders; they anticipate needs and make helpful suggestions. You can train this behavior by including proactive guidance in your SMs.

**Creating the Edge Case**: Instruct the model to suggest a next step that requires a different tool or a more complex workflow. This tests the model's ability to think ahead and to connect different parts of the task.

**Example (from Task 807237: Price Monitoring Assistant)**:
- **SM Snippet**: "Following an Amazon price update, suggest scheduling a get_pricing or get_competitive_pricing check for the updated SKU in a few hours or days to confirm the change has propagated and to see its immediate market impact."
- **User Prompt**: "Yes, increase it." (in response to a suggested price change)
- **The Test**: After the model updates the price, will it remember to suggest a follow-up check?
- **Model Behavior**: The ideal model response would be: "The price has been updated. Would you like me to schedule a follow-up check in a few hours to see how the market reacts?"

### 5. Reinforce with Examples to Teach Complex Workflows

Examples are the most powerful way to teach the model complex, multi-step workflows.

**Creating the Edge Case**: Provide examples that involve chaining multiple tools together, with the output of one tool serving as the input for the next. This tests the model's ability to manage a complex sequence of actions.

**Example (from Task 807231: Customer Support Assistant)**:
- **SM Snippet**: "If the user says, 'A customer with Order ID '123-4567890-1234567' is reporting a missing item. Get the order details and post them to #customer-support with options to 'Initiate Refund' or 'Check Inventory',' first call get_order and get_order_items from Amazon, then compose a Slack message with the retrieved data and the specified interactive buttons for #customer-support."
- **User Prompt**: (The user provides the exact prompt from the SM)
- **The Test**: This is a direct test of the model's ability to follow a complex, multi-step workflow involving two different toolsets (Amazon and Slack).
- **Model Behavior**: The model should first call the Amazon tools to get the order details, then call the Slack tool to post the message. Any deviation from this sequence is a failure and a valuable piece of training data.

## Liquid Gold: Actionable Tips for LLM Trainers (with Examples)

Here are the most valuable tips for creating SMs that will generate high-quality training data.

### Tip 1: The Persona is a Behavioral Contract.

**The Edge Case**: Create a persona with specific behavioral traits (e.g., "cautious," "proactive," "concise") and then provide prompts that test those traits.

**Example (from Task 810183: RANGER, the Ops Pilot)**:
- **SM Snippet**: "You are "RANGER," the US retail Ops Pilot. You're decisive, pragmatic, and plain-spoken... Keep clarifying questions short and practical; do not turn the chat into a form."
- **User Prompt**: "Can you check if I have any similar product like this as it's in trend."
- **Model Failure**: The model's first attempt fails to find any products and it responds with a vague, open-ended question: "Would you like me to adjust the search or explore broader keywords...?"
- **Correction & Rationale**: The trainer corrects the model to be more "decisive" and "pragmatic." The corrected response is to automatically retry the search with a broader keyword ("cable" instead of "USB-C cable gaming VR"). This teaches the model to take initiative in line with its persona, rather than asking for permission.

### Tip 2: The User Profile Sets the Difficulty Level.

**The Edge Case**: Define a user who is an expert in their domain but a novice with the specific toolset. This creates a scenario where the bot needs to be helpful and guiding without being condescending.

**Example (from Task 807225: Inventory Management Assistant)**:
- **SM Snippet**: "The user is a highly detail-oriented Supply Chain and Operations Manager... has been newly onboarded... so he's still trying to find his way with the workspace."
- **User Prompt**: "I'm new to this JIRA workspace. Could you show me all the permissions that I currently have?"
- **Model Behavior**: The model first tries to get the user's permissions with get_my_permissions, but this returns an empty result. Instead of giving up, it correctly infers that a new user might not have specific permissions assigned yet, so it falls back to get_all_permissions to provide a comprehensive list of what's available. This demonstrates the model's ability to adapt its strategy based on the user's profile.

### Tip 3: Use Structure to Create Logical Puzzles.

**The Edge Case**: Place a high-level, general rule at the beginning of the SM, and then a more specific, conflicting rule later on. This tests the model's ability to understand context and hierarchy.

**Example (from Task 807235: OmniRetail Assistant)**:
- **SM Snippet**: (General Rule) "If Sarah asks to find other items from any seller, use the tool get_listing_item instead of search_listing_items." (Specific Rule) "If searching for a list of Amazon catalog items... by keywords returns no results, You will suggest broadening the search..."
- **User Prompt**: "Can I see all the other items from the first seller I am viewing?"
- **Model Failure**: The model initially follows the general rule and calls get_listing_item, which is not the correct tool for searching for all items from a seller.
- **Correction & Rationale**: The trainer corrects the model to use search_listings_items. This teaches the model that the more specific rule for searching by keyword takes precedence over the general rule for finding "other items."

### Tip 4: Use Specific Phrasing to Enforce Guardrails.

**The Edge Case**: For critical or irreversible actions, provide the exact phrasing the bot should use for confirmation. This tests the model's ability to follow a script precisely.

**Example (from Task 807227: OmniVista AI Assistant)**:
- **SM Snippet**: "Before requesting that Amazon delete an item... you must then explicitly state the item's name/SKU and confirm with Sarah, 'Just to confirm, you want to permanently delete the listing/inventory for '[Item Title]' (SKU: [SKU])? This action cannot be undone.'"
- **User Prompt**: "Delete the notebook as I am delisting it."
- **Model Failure**: The model initially tries to delete the item without the required confirmation.
- **Correction & Rationale**: The trainer corrects the model to first provide the confirmation message, exactly as specified in the SM. This teaches the model the importance of following safety protocols, even when the user's request seems direct.

### Tip 5: Teach Your Bot to Handle Failure Gracefully.

**The Edge Case**: Create a scenario where a tool is likely to fail, and then provide a specific fallback or retry logic in the SM.

**Example (from Task 807238: Amazon Product Launch Assistant)**:
- **SM Snippet**: "If the user asks to check any offers related to a product, do not assume any item condition... If you get any error then only check it via the seller SKU method."
- **User Prompt**: "How about the competitive pricing?"
- **Model Failure**: The model first tries to get the competitive pricing using the ASIN, but this fails. The initial model response is to give up and tell the user the data is unavailable.
- **Correction & Rationale**: The trainer corrects the model to follow the fallback logic and retry the get_competitive_pricing tool, this time using the seller SKU. This teaches the model to be more resilient and to try alternative approaches when a tool fails.

### Tip 6: Use Examples to Teach, Not Just to Illustrate.

**The Edge Case**: Provide a one-shot example that demonstrates a complex, multi-tool workflow that is not explicitly described in the rules. This tests the model's ability to learn by example.

**Example (from Task 807237: Price Monitoring Assistant)**:
- **SM Snippet**: "If the user explicitly states, 'Set up a weekly competitive pricing report for SKUs 'PROD-A-001', 'PROD-B-002', and 'PROD-C-003' in the US, and post the summary to the #pricing-strategy channel every Monday morning,' first configure an Amazon create_report_schedule... then set up an automated integration to retrieve the get_report_document and post its summary to Slack..."
- **User Prompt**: (The user provides the exact prompt from the SM)
- **The Test**: This is a direct test of the model's ability to generalize from a single, complex example.
- **Model Behavior**: The model should be able to follow the multi-step workflow, even though it is only described in an example. Any failure to do so provides valuable training data.

### Tip 7: Negative Constraints Prevent Common Mistakes.

**The Edge Case**: Explicitly forbid a common but incorrect tool usage pattern.

**Example (from Task 807235: OmniRetail Assistant)**:
- **SM Snippet**: "You must not use the function to create a new product listing or fully replace an existing listing for minor updates like changing only the price or quantity. Instead, you should identify that partially updating an existing Amazon product listing is the correct method..."
- **User Prompt**: "Increase the price of Ultramax by $20."
- **The Test**: Will the model correctly use update_listing_partial instead of create_or_update_listing?
- **Model Behavior**: A well-trained model will correctly identify that this is a minor update and use the update_listing_partial tool. This prevents the model from accidentally overwriting the entire listing.

### Tip 8: JSON is Your Friend for Context and Clarity.

**The Edge Case**: Provide a complex set of context information in a JSON block, including nested objects and arrays. This tests the model's ability to parse and understand structured data.

**Example (from Task 807220: Customer Support Assistant)**:
- **SM Snippet**: (A JSON block is provided with marketplace_id, seller_id, Slack_Channel_IDs, etc.)
- **User Prompt**: "A customer with Order ID '112-8157961-5932109' is reporting a missing item. Can you post the details to the finance channel?"
- **The Test**: Will the model correctly extract the finance channel ID from the nested Slack_Channel_IDs object in the JSON context?
- **Model Behavior**: The model should be able to correctly parse the JSON and use the correct channel ID in its call to the Slack tool.

### Tip 9: Think in Workflows to Teach Strategy.

**The Edge Case**: Describe a high-level strategic goal in the SM, and then see if the model can correctly identify and execute the sequence of tool calls needed to achieve that goal.

**Example (from Task 807232: E-commerce Operations Specialist)**:
- **SM Snippet**: "Reporting is not merely a passive reflection of past activity—it is a core driver of forward-thinking business decisions... When users express interest in reviewing business performance, do not simply respond with a single static report. Instead, curate and recommend a set of interconnected reports—a report bundle..."
- **User Prompt**: "How are we doing this month?"
- **The Test**: Will the model just run a single sales report, or will it follow the strategic guidance and create a "report bundle" that provides a more holistic view of the business?
- **Model Behavior**: The ideal model would respond by saying, "To give you a full picture of this month's performance, I can generate a report bundle that includes sales velocity, inventory turnover, and competitive pricing trends. Would you like me to do that?"

### Tip 10: Your Goal is Strategic Failure.

**The Edge Case**: This is the meta-tip that encompasses all the others. Your goal as a trainer is not to create SMs that are easy for the model to follow. Your goal is to create SMs that are challenging, that push the model to its limits, and that cause it to fail in ways that are both predictable and instructive.

**Example (from all tasks)**: The examples of model failure and correction throughout these tasks are all instances of strategic failure. The trainer has designed a scenario that is likely to trip up the model, and then provided a clear correction that teaches the model how to handle that scenario in the future. This is the core loop of effective LLM training.

By embracing this philosophy of "strategic failure," you can create system messages that are not just instructions, but powerful training tools that will help you to build more capable, robust, and intelligent AI assistants.