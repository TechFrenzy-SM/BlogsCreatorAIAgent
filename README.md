# Blog Creator Agent

This project is about creating blog posts using AI Agents. The framework used here is Autogen 0.4, and the LLM model used is GPT-4o hosted on Azure AI Foundry. The code is written in Python3.

| Framework   | Model  | Platform         | Language |
| ----------- | ------ | ---------------- | -------- |
| Autogen 0.4 | GPT-4o | Azure AI Foundry | Python3  |

## Security

This sample uses access tokens from Entra ID to enhance security instead of using API keys. Access tokens provide a more secure and flexible way to authenticate and authorize access to resources. By using Entra ID, we ensure that the tokens are securely managed and can be easily rotated or revoked if necessary. This approach reduces the risk of exposing sensitive API keys and helps maintain a higher level of security for the application.

## Code Overview

### Create the Primary Agent

The primary agent, `Story_writer`, is responsible for writing technical blog posts. It uses the following configuration:

```python
Story_writer = AssistantAgent(
    name="Story_writer",
    model_client=client,
    system_message="""You are a helpful AI assistant that writes technical blog posts.
    You can help them with story building, sharing real facts and figures, examples, snippets, and other useful links.
    A code snippet should only be added if the story needs it. Keep the blog posts engaging and short within 300 words.
    """,
)
```

### Create the Critic Agent

The critic agent, `Story_reviewer`, provides constructive feedback on the technical blog posts to ensure they stick to the topic and provide relevant facts, figures, examples, and correct code snippets. It uses the following configuration:

```python
Story_reviewer = AssistantAgent(
    name="Story_reviewer",
    model_client=client,
    system_message="""You are a helpful AI assistant which provides constructive feedback on the technical blog posts to make sure they stick to the topic, and provide relevant facts, figures, examples, and correct code snippets that align to the topic.
    Make sure the blogs have great story-telling and check if the blog content needs a code snippet or not. Only if a code snippet is needed, it should be added.
    Respond with 'APPROVE' when your feedbacks are addressed.
    """,
)
```

### Define Termination Condition

A termination condition is defined to stop the task if the critic approves:

```python
text_termination = TextMentionTermination('APPROVE')
```

### Create a Team with the Primary and Critic Agents

A team is created with the primary and critic agents:

```python
team = RoundRobinGroupChat([Story_writer, Story_reviewer], termination_condition=text_termination)
```

### Main Function

The main asynchronous function in this project generates a blog post on the topic of "Data Governance for AI" and streams the messages to the console. If the output contains the stop reason "APPROVE", the blog post content is saved to a markdown file in the `./outputs` directory.

```python
# Define the main asynchronous function
async def main() -> None:
    # Stream the messages to the console
    output = await Console(
        team.run_stream(task="Write a blog post of Data Governance for AI")
    )

    print(f"Final Output: {output.messages[-2].content}")
    print(f"\nOutput_Stop_Reason: {output.stop_reason}")

    # Check if output contains stop_reason "APPROVE"
    if "APPROVE" in output.stop_reason:
        print("\nBlog post approved")
        blog_post_content = output.messages[-2].content

        if blog_post_content:
            print(f"\nBlog_Post_Contents: {blog_post_content}")
            print(f"\nBlog_Post_Length: {len(blog_post_content)}")

            # Extract the blog post title
            title_match = re.search(r'\*\*(.*?)\*\*', blog_post_content)
            if title_match:
                blog_post_title = title_match.group(1).split(':')[0].replace(' ', '_')
                print(f"\nBlog_Post_Title: {blog_post_title}")
                filePath = os.path.join("outputs", f"{blog_post_title}.md")
                print(f"\nFile_Path: {filePath}")

                # Ensure the output directory exists
                os.makedirs("outputs", exist_ok=True)

                # Save the output to a .md file in ./outputs directory
                with open(filePath, "w", encoding="utf-8") as f:
                    f.write(f"# {title_match.group(1)}\n\n")
                    f.write(blog_post_content)
                print(f"\nFile created at: {filePath}")
            else:
                print("\nTitle not found in the blog post content.")
        else:
            print("\nNo blog post content generated")
    else:
        print("\nBlog post not approved")

# Check if there's an existing event loop
try:
    loop = asyncio.get_running_loop()
except RuntimeError:
    loop = None

if loop and loop.is_running():
    # If there's an existing event loop, use it to run the main coroutine
    asyncio.ensure_future(main())
else:
    # Otherwise, use asyncio.run()
    asyncio.run(main())
```

This code ensures that the blog post content is properly generated, approved, and saved to a markdown file with the appropriate title.
