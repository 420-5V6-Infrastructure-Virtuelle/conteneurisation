---
title: "Bug en Production - Code IA"
weight: 1
---

# Roleplay Script: Production Bug from AI-Generated Code

## Module 10 - Conventions d'Équipe et Tensions

### Scenario Overview

**Duration:** 15 minutes
**Participants:** 4-6 people in a team
**Roles:**
- **Lead Dev** (Senior, skeptical of AI tools)
- **Junior Dev** (Alice, enthusiastic AI tool user)
- **QA Engineer** (Bob, discovered the bug)
- **Team Lead** (Manager, focused on process)
- **Optional:** Product Owner, Security Engineer

**Context:** A critical bug appeared in production on Friday afternoon. Money-related functionality is affected.

---

## Act 1: Discovery - "Friday 4:47 PM" (3 min)

### Bob (QA Engineer) enters Slack:

```
#incidents channel

Bob @channel We have a critical issue in prod.
Users are seeing wrong order amounts on the confirmation page.
The actual charge is correct, but the display is broken.
Example: Order #4523 shows $45.00 confirmation, actual charge was $4,500.00
```

**Team Lead:** "How many users affected?"

**Bob:** "Checking logs... 147 orders in the last 24 hours have this discrepancy. All of them are high-value orders (> $500). For small orders, it works fine."

**(Optional) Product Owner enters:**

```
Customer Support just forwarded me a ticket. 
A customer is threatening legal action because they thought 
they were charged $67 but were actually charged $6,700.

This is a BIG problem. We need answers NOW.
```

---

## Act 2: Investigation - "Friday 5:15 PM" (5 min)

### Lead Dev takes charge:

**Lead Dev:** "Let me see the PR that introduced this. Bob, which commit?"

**Bob:** "The bug started appearing after deploy v2.3.4. That's commit 8f7a2bc."

**Lead Dev:** `git show 8f7a2bc --stat`

```
commit 8f7a2bc3e4d5f6789012345678901234567890ab
Author: Alice <alice@company.com>
Date:   Thu Jan 18 14:32:00 2024

    feat: improve order confirmation display
    
    - Refactored price formatting logic
    - Added decimal handling for better UX
    - Cleaned up unused test cases

    3 files changed, 45 insertions(+), 78 deletions(-)
```

**Lead Dev:** `git show 8f7a2bc src/services/format_price.py`

```python
# src/services/format_price.py

def format_order_amount(amount: float) -> str:
    """
    Format order amount for display.
    
    Args:
        amount: Order amount in dollars
        
    Returns:
        Formatted string for UI display
    """
    # IMPROVEMENT: Moving decimal for better readability
    # AI suggestion: shows prices more consistently
    display_amount = amount / 100
    
    return f"${display_amount:.2f}"
```

**Lead Dev:** "What... the... hell. Why is the amount divided by 100?"

**Alice (Junior Dev):** "The AI suggested that change. It said it was 'standard practice' for financial displays."

**Bob:** "Let me check... the input to this function comes from the database."

```python
# Existing code that calls this function
order_amount = get_order_total(order_id)  # Returns amount in CENTS
display_text = format_order_amount(order_amount)
```

**Bob:** "Found it. The order total from DB is in CENTS. The original function was just `return f"${amount:.2f}"`. The AI added the divide by 100, creating a 'cents to dollars' conversion that was already done."

**Impact:**
- $450,000 cents / 100 / 100 = $45 (wrong display)
- $450,000 cents / 100 = $4,500 (correct display)

**Lead Dev:** "Wait, why do we have amount in cents in one place and dollars elsewhere? That's inconsistent."

**Alice:** "I asked the AI to 'improve consistency'. It said standardizing to one unit would be better. But it only changed the display function, not the callers."

**Team Lead:** "How did this get merged without anyone noticing?"

**Bob:** "Let me check the PR review..."

---

## Act 3: The PR Review - "What Happened?" (5 min)

### PR Review Thread:

```
#pull-requests channel - Wed Jan 17

Alice opened PR #234: "feat: improve order confirmation display"

Lead Dev: Alice, I don't have time for a full review today. 
          Can this wait until Monday?

Alice: The AI said it's a clean refactor. Just standardizing price formatting.
       Passed all tests locally. Can we merge so I can move on to the next task?

Lead Dev: [sighs] Fine. Let me do a quick skim...
         LGTM. But please add more tests next time.
         Approved and merged.

Alice: Thanks!
```

### Reality check:

**Lead Dev:** "I approved it... I didn't have time for a proper review."

**Alice:** "The AI wrote it so quickly, I assumed it was simple. And all tests passed!"

**Team Lead:** "Let me see the tests."

**Bob:** `git show 8f7a2bc tests/test_format_price.py`

```python
# tests/test_format_price.py

# REMOVED: Test cases for edge conditions (AI: "redundant tests")
# - test_format_price_large_value
# - test_format_price_decimal_places  
# - test_format_price_matches_database_value

# KEPT: Only basic tests
def test_format_price_returns_string():
    assert isinstance(format_order_amount(100.0), str)

def test_format_price_has_dollar_sign():
    result = format_order_amount(100.0)
    assert "$" in result
```

**Lead Dev:** "The AI deleted the tests that would have caught this! And I approved it without noticing."

**Alice:** "The AI said those tests were 'redundant' because they tested the same function. I didn't question it..."

---

## Act 4: The Debrief - "Whose Fault Is It?" (2 min)

### Tension Points:

**Lead Dev:** "Alice, you should have verified the AI's changes. You can't blindly trust it."

**Alice:** "I did read the code! It made sense. And YOU approved it without a proper review. You're the senior."

**Team Lead:** "This isn't about blame. It's about process. What failed?"

**Bob:** "Multiple things failed. The deleted tests. The missing integration test. The rushed review. The AI confidently making wrong suggestions."

**Potential responses to explore:**

1. **Alice's responsibility:** Did she understand the code she committed?
2. **Lead Dev's responsibility:** Did he review properly?
3. **Team process:** No one caught the test deletion?
4. **AI tooling:** How to catch AI-generated bugs before production?

---

## Discussion Questions (Post-Roleplay)

### For the team to discuss:

1. **Code Ownership**
   - If Alice committed the code, is she responsible?
   - If Lead Dev approved, is he responsible?
   - Does "AI wrote it" change anything?

2. **Testing Practices**
   - Why weren't there integration tests for price display?
   - How can we catch test deletions that shouldn't happen?
   - Should AI be allowed to delete tests?

3. **Review Culture**
   - "LGTM" vs actual review - when is it acceptable?
   - How to handle time pressure during reviews?
   - Should AI-generated code require different review?

4. **Process Gaps**
   - No one noticed the pattern of deleted tests?
   - Is our code review checklist sufficient?
   - How to integrate AI skepticism into team culture?

5. **Going Forward**
   - What rules should we add to our charter?
   - How do we prevent this in the future?
   - Is our team mature enough for AI tools?

---

## Charter Drafting Exercise (15 min)

Based on the roleplay, have teams draft sections for their AI usage charter:

### Section: Code Review Standards

```markdown
## Code Review Standards

### AI-Generated Code
- Must be fully understood by the committer
- Test deletions require explicit justification
- Changes to financial/security code require 2-person review

### Merge Requirements
- At least one comprehensive review (not just "LGTM")
- All CI checks pass
- Integration tests for critical paths
```

### Section: Testing Requirements

```markdown
## Testing Requirements

### Critical Functionality
- Price calculations: MUST have integration tests matching DB values
- Security: MUST have security-focused tests
- Data integrity: MUST have tests with realistic data

### Test Deletions
- Never delete tests without explicit justification
- AI suggestions to delete tests: ALWAYS question
- If test is genuinely obsolete: Comment with reason
```

### Section: Team Communication

```markdown
## Team Communication

### When Using AI Tools
- Announce when code is AI-generated
- Highlight areas you're uncertain about
- Ask for second opinion on complex AI suggestions

### During Reviews
- Flag AI-generated code explicitly
- Extra scrutiny for deleted tests/code
- Financial and security changes: require domain expert
```

---

## Roleplay Tips for Facilitator

### Before starting:
- Assign roles randomly or let people choose
- Give each role a private brief with their motivations
- Set a 15-minute timer

### During roleplay:
- Let tension build naturally
- Don't intervene unless discussion stalls
- Note key points for debrief

### After roleplay:
- Have everyone step out of character
- Start with "How did that feel?"
- Guide discussion toward concrete charter items

### Common insights:
- Everyone thought someone else would catch the bug
- AI confidence made humans less skeptical
- Time pressure led to shortcuts
- Testing gaps were organizational, not technical