---
Author: Hitesh Bandhu
Email: bandhuhitesh52003@gmail.com
Twitter: https://x.com/hitmadeit
Youtube: https://youtube.com/@hiteshbandhucodes
Website: https://www.hiteshbandhu.xyz
---
---
***21 Jun 2024***
##### tried crew ai today, to make an ai agent that uses just the standard python library and nothing outside that, here is the website for docs
[link to docs](https://docs.crewai.com/core-concepts/Agents/)

###### here is the code to review, and my own prompts that i used, very basic, will use advanced prompts now
 
```python
from dotenv import load_dotenv
load_dotenv()

from crewai import Agent, Task, Crew
from crewai_tools import PDFSearchTool

code_agent = Agent(
  role='Programmer',
  goal='Code a Snake Game in Python',
  backstory="""You're a python developer specialising in developing small arcade games just using the python documentaion, and not using any internet. You just use the PDF you have access to - to read the documentation and code the games.""",
  verbose=True
)

pdf_tool = PDFSearchTool("library.pdf")

task = Task(
  description='Code a Snake Game in Python',
  expected_output='Python Code for a simple Snake Game.',
  agent=code_agent,
  tools=[pdf_tool]
)

crew = Crew(
    agents=[code_agent],
    tasks=[task],
    verbose=True
)

result = crew.kickoff()
print(result)
```

---


