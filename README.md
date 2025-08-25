# Logic App Evaluation Exam — Customer Data Validation

## Overview
This project demonstrates an **Azure Logic App (Standard)** workflow that integrates with a custom **.NET assembly** for customer data validation.  
The workflow validates email addresses and phone numbers submitted via HTTP request, using validation logic encapsulated in a **.NET class library (DLL)** and exposed through **Azure Functions**.

---

## Project Structure
```
C:\src\LogicAppValidation
│
├── CustomerValidationLib/        # .NET class library with validation logic
│   ├── Validator.cs
│   └── ValidationResult.cs
│
├── CustomerValidationApi/        # Azure Functions project (in-process)
│   ├── ValidateEmail.cs
│   ├── ValidatePhoneNumber.cs
│   └── CustomerValidationApi.csproj
│
└── CustomerValidationApp/        # Logic App Standard project
    └── Workflows/
        └── ValidateCustomer/
            └── workflow.json
```

---

## Prerequisites
- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools v4](https://github.com/Azure/azure-functions-core-tools)
- [Azurite](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) (local storage emulator)
- [VS Code](https://code.visualstudio.com/) with **Logic Apps (Standard)** extension

---

## How to Run Locally

### 1. Build the Validation Library
```powershell
cd CustomerValidationLib
dotnet build -c Release
```

### 2. Run the Functions API
```powershell
cd ..\CustomerValidationApi
func start
```

Endpoints will be available:
- `POST http://localhost:7071/api/ValidateEmail`
- `POST http://localhost:7071/api/ValidatePhoneNumber`

Test directly:
```powershell
Invoke-RestMethod -Method POST -Uri "http://localhost:7071/api/ValidateEmail" `
  -Headers @{ "Content-Type"="application/json" } `
  -Body '{"email":"john.doe@example.com"}'
```

---

### 3. Run the Logic App
In a **second terminal**:

```powershell
cd ..\CustomerValidationApp
func start --port 7072
```

Trigger URL:
```
http://localhost:7072/runtime/webhooks/workflow/api/management/triggers/http_in?api-version=2020-05-01-preview&workflowName=ValidateCustomer
```

---

### 4. Test End-to-End
Send input to the Logic App:

```powershell
Invoke-RestMethod -Method POST `
  -Uri "http://localhost:7072/runtime/webhooks/workflow/api/management/triggers/http_in?api-version=2020-05-01-preview&workflowName=ValidateCustomer" `
  -Headers @{ "Content-Type"="application/json" } `
  -Body '{
    "customerId":"12345",
    "email":"john.doe@example.com",
    "phoneNumber":"123-456-7890"
  }'
```

✅ Expected Success Response:
```json
{
  "customerId": "12345",
  "emailValidation": {
    "isValid": true,
    "message": "Email is valid"
  },
  "phoneValidation": {
    "isValid": true,
    "message": "Phone number is valid"
  },
  "status": "Success"
}
```

 Invalid inputs return HTTP 400 with status `"Error"`.

---

## Notes & Assumptions
- Functions are implemented using **in-process model** for simplicity.
- Regex supports basic US-style phone numbers.
- Local storage emulator (Azurite) is required for Logic Apps Standard runtime.
- Solution tested with **.NET 8**, **Azure Functions Core Tools v4**.

---

## Deliverables
- Custom .NET assembly for validation logic
- Functions API exposing validation endpoints
- Logic App workflow calling the API and shaping response
- Sample test cases included in this README
