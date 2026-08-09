
- ## Question 1:
- ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the process that created remote threads in rundll32.exe. Answer format: _.exe
## Solution:
- Firstly, I identified event id for creating remote threads, which is event id 8, and the .exe file in which remote threads were created was also given, only the sourceimage was needed to complete the research:
  ```
  index="main" sourcetype="WinEventLog:Sysmon" EventCode=8 TargetImage=*rundll32.exe | stats count by SourceImage
  ```
    
- ## Question 2:
    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the process that started the infection. Answer format: _.exe
    
    ## Solution:
    - I couldn't find any clue after running a lot of query so I fetch a query which was:
      ```
      index=* EventCode=1 
| eval exe = replace(Image,".*\\\\","")
| stats earliest(_time) AS first_seen BY exe, Image, CommandLine
| sort 0 + first_seen
| table first_seen exe Image CommandLine
      ```
    - **Answer:** rundll32.exe
