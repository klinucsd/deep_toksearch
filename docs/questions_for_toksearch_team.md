# Questions for TokSearch Team Meeting

**Goal:** Improve the AI assistant's understanding of TokSearch so domain scientists can use it through natural language without needing to code.

---

## Understanding Common Workflows

1. **What are the most common tasks scientists do with TokSearch?**
   - Example: Analyze EFIT data, compare shots, find specific patterns
   - Goal: Understand typical workflows to include in skills

2. **What are the typical shot ranges you work with?**
   - Example: Recent experiments (shots 170000-180000), specific campaigns
   - Goal: Use realistic shot numbers in examples and requests

3. **What are the most frequently used signals?**
   - Which trees, which specific signals do scientists fetch most often?
   - Goal: Focus skills on the most important signals

---

## Signal Information

4. **Are there any signal names that commonly confuse users?**
   - Example: We found `\ipmeas` vs `\ip` confusion in magnetics tree
   - Goal: Add warnings to skills for common mistakes

5. **What are the correct signal expressions for common diagnostics?**
   - Please provide a list of the top 10-20 most-used signals with correct syntax
   - Format: `MdsSignal(r'\EXPRESSION', 'TREENAME')`
   - Goal: Build a comprehensive signal reference table

6. **Are there any "hidden" or less-documented signals that are useful?**
   - Signals that experienced users know about but aren't well documented
   - Goal: Uncover valuable signals that should be in the skills

---

## Data Patterns & Units

7. **What are the typical units for different signal types?**
   - Which signals return data in Amps vs MA, eV vs keV, etc.
   - Goal: Teach the AI about unit conversions

8. **What are typical value ranges for key parameters?**
   - Example: Plasma current usually 0.5-2.0 MA, qmin typically 1-4, etc.
   - Goal: Help the AI spot unusual values or potential data issues

9. **Are there any signals that return different data structures?**
   - Some signals might return scalars, 2D profiles, or other formats
   - Goal: Document all data format variations

---

## Common Pitfalls

10. **What are the most common mistakes users make?**
    - Help us add warnings and "common mistakes" sections to the skills

11. **Are there any "gotchas" in the API that trip people up?**
    - Non-obvious behaviors, edge cases, or counterintuitive features

12. **What errors do new users frequently encounter?**
    - Help us add better error handling guidance

---

## Documentation Gaps

13. **What information is missing from current docs that users always ask about?**
    - Frequently asked questions or concepts that need better explanation

14. **Are there any examples you wish every new user would see first?**
    - "Hello world" examples that should be prominently featured

---

## Testing & Development

15. **Do you have sample datasets or test shots we can use for mock data?**
    - Real or realistic test data would make mock data generation more useful
    - Goal: Make mock data more representative of real data

16. **What's a good "hello world" example for new users?**
    - A simple, meaningful example that demonstrates TokSearch's value
    - Goal: Create a compelling first example for beginners

---

## Domain-Specific Knowledge

17. **What domain knowledge should the AI know to help scientists?**
    - Example: What "qmin > 2" means physically, why certain thresholds matter
    - Goal: Help the AI understand the context and provide better assistance

18. **What physical constraints or limitations should the AI be aware of?**
    - Example: Maximum achievable values, physical limits of the device
    - Goal: Help the AI spot unrealistic or erroneous data

---

## Workflow Questions

19. **Can you describe a complete analysis workflow from start to finish?**
    - From "I want to analyze X" to final results
    - Example question: "Show me all shots where qmin > 2 and ipmhd > 1.5 MA"
    - Goal: Understand the typical scientist workflow

20. **What questions do scientists typically ask about their data?**
    - Help the AI understand what kinds of queries to expect
    - Example: "Which shots had the highest plasma current?", "Find shots similar to this one"

---

## Code Quality & Best Practices

21. **Are there any naming conventions or coding standards you recommend?**
    - For variable names, function names, file organization

22. **What makes a TokSearch script "good" vs "bad" in your view?**
    - Characteristics of maintainable, readable, robust code

---

## Notes

- The goal is to enable scientists to use TokSearch through natural language without writing code
- We've created 4 skills for the AI assistant covering basics, signals, backends, and DIII-D specifics
- Two test requests have been successfully completed - we can share the results
- Your expertise will help make the AI assistant more useful for the fusion science community

Thank you for your time and expertise!
