# Airline Helpdesk Chatbot: Execution Flow Diagram

## Customer Interaction Flow

```
┌─────────────────┐     ┌─────────────────────┐     ┌───────────────────┐
│                 │     │                     │     │                   │
│  Customer       │──►  │  Web Interface      │──►  │  Neuro-SAN Server │
│  Question       │     │  (Port 5001)        │     │  (Port 30011)     │
│                 │     │                     │     │                   │
└─────────────────┘     └─────────────────────┘     └───────────┬───────┘
                                                               │
                                                               ▼
┌─────────────────┐     ┌─────────────────────┐     ┌───────────────────┐
│                 │     │                     │     │                   │
│  Final Response │◄──  │  Response           │◄──  │  Airline 360      │
│  to Customer    │     │  Compilation        │     │  Assistant        │
│                 │     │                     │     │                   │
└─────────────────┘     └─────────────────────┘     └───────────┬───────┘
                                                               │
                                                               ▼
                                                   ┌───────────────────────┐
                                                   │                       │
                                                   │  Domain Selection     │
                                                   │  Process              │
                                                   │                       │
                                                   └──┬──────────┬─────────┘
                                                      │          │
                        ┌─────────────────────────────┘          └────────────────────────────┐
                        │                                                                     │
                        ▼                                                                     ▼
          ┌─────────────────────────┐                                             ┌─────────────────────────┐
          │                         │                                             │                         │
          │  Baggage_Handling       │                                             │  Flights                │
          │  Agent                  │                                             │  Agent                  │
          │                         │                                             │                         │
          └───┬─────────┬─────┬─────┘                                             └────┬────────┬───────────┘
              │         │     │                                                        │        │
      ┌───────┘    ┌────┘     └───────┐                                       ┌────────┘        └───────┐
      │            │                  │                                       │                         │
      ▼            ▼                  ▼                                       ▼                         ▼
┌───────────┐ ┌───────────┐    ┌───────────┐                          ┌───────────┐             ┌───────────┐
│           │ │           │    │           │                          │           │             │           │
│ Carry_On  │ │ Checked   │    │ Special   │                          │ Military  │             │ Mileage   │
│ Baggage   │ │ Baggage   │    │ Items     │                          │ Personnel │             │ Plus      │
│           │ │           │    │           │                          │           │             │           │
└───────────┘ └───────────┘    └───────────┘                          └───────────┘             └───────────┘
```

## Detailed Agent Processing Flow

When a customer question is received, the processing follows this sequence:

```
1. INQUIRY ANALYSIS
   ┌───────────────────────┐
   │ Top-level agent       │
   │ analyzes inquiry      │◄─── Customer question
   └───────────┬───────────┘
               │
               ▼
2. DOMAIN IDENTIFICATION
   ┌───────────────────────┐
   │ Determine which       │
   │ domain(s) are relevant│
   └───────────┬───────────┘
               │
               ▼
3. REQUIREMENT COLLECTION
   ┌───────────────────────┐
   │ Ask sub-agents what   │
   │ information they need │
   └───────────┬───────────┘
               │
               ▼
4. USER FOLLOW-UP (if needed)
   ┌───────────────────────┐
   │ Ask customer for      │
   │ missing information   │──► Customer provides additional info
   └───────────┬───────────┘
               │
               ▼
5. DELEGATION
   ┌───────────────────────┐
   │ Route inquiry to      │
   │ specialized agents    │
   └───────────┬───────────┘
               │
               ▼
6. INFORMATION RETRIEVAL
   ┌───────────────────────┐
   │ Extract relevant      │
   │ policy information    │
   └───────────┬───────────┘
               │
               ▼
7. RESPONSE FORMULATION
   ┌───────────────────────┐
   │ Create detailed       │
   │ response with URLs    │
   └───────────┬───────────┘
               │
               ▼
8. RESPONSE COMPILATION
   ┌───────────────────────┐
   │ Combine and refine    │
   │ all agent responses   │
   └───────────┬───────────┘
               │
               ▼
9. FINAL RESPONSE
   ┌───────────────────────┐
   │ Present comprehensive │
   │ answer to customer    │──► Final response to customer
   └───────────────────────┘
```

## Tools Invocation Flow

```
Specialized Agent
      │
      ├─────────────────┬──────────────────┐
      │                 │                  │
      ▼                 ▼                  ▼
┌───────────┐    ┌────────────┐    ┌─────────────┐
│ ExtractPdf │    │ URLProvider │    │ Other Tools │
└─────┬─────┘    └──────┬─────┘    └──────┬──────┘
      │                 │                  │
      ▼                 ▼                  ▼
┌──────────────────────────────────────────────────┐
│                                                  │
│               Response Compilation               │
│                                                  │
└──────────────────────────────────────────────────┘
```

## Example Question Flow

For a customer inquiry about checked baggage weight limits:

1. **Initial Inquiry**: "What is the maximum weight allowed for checked baggage on international flights?"

2. **Top-Level Processing**:
   - Airline 360 Assistant identifies this as a baggage question with international aspects
   - Routes to both Baggage_Handling and International_Travel agents

3. **Domain Processing**:
   - Baggage_Handling routes to Checked_Baggage
   - International_Travel routes to International_Checked_Baggage

4. **Information Retrieval**:
   - Both agents use ExtractPdf to read relevant policy documents
   - Agents extract weight limit information for international flights

5. **Response Compilation**:
   - Information about weight limits is combined
   - URLProvider adds links to relevant policy pages
   - Comprehensive response is formulated with all aspects covered

6. **Final Response**: Customer receives complete information about international checked baggage weight limits with relevant URLs