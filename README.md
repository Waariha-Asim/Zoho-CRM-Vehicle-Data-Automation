# 🚗 Zoho CRM Vehicle Data Automation

> **Automated vehicle-data collection and CRM lifecycle automation using n8n, Zoho CRM, Zoho Forms, and Gmail.**

**Zoho Crm Vehicle Data Automation** is an end-to-end automation built for a **German insurance brokerage workflow**. The solution connects Zoho CRM, Zoho Forms, n8n, and Gmail to collect missing vehicle information from customers and automatically write the answers back to the **correct Fahrzeug record** — without manual data entry.

The complete CRM, form, status flow, and customer-facing communication are configured in **German** to match the intended business environment.

---

## 📊 Workflow Output

![Workflow Output](https://github.com/Waariha-Asim/Zoho-CRM-Vehicle-Data-Automation/blob/main/Workflow_Output.png)

The solution consists of two connected n8n workflows covering the complete customer-data round trip.

---

## 📩 1. Vehicle Data Request Workflow

The process starts when an employee opens a **Fahrzeug** record in Zoho CRM and clicks the **„Datenabfrage senden“** button.

**Flow:**

`Zoho CRM Button → n8n Webhook → Retrieve Fahrzeug & Kontakt → Build Personalized Form URL → Gmail → Update CRM → Wait → Check Status → Reminder`

The workflow:

1. Receives the selected Fahrzeug record ID from the CRM button.
2. Retrieves the vehicle and its linked customer/contact.
3. Builds a personalized **Zoho Form URL** containing:

   * `FahrzeugID` as a hidden identifier
   * `Kennzeichen` as a prefilled read-only field
   * `Fahrzeug` as a prefilled read-only field
4. Sends the customer a short, friendly **German email** containing the form link.
5. Updates the Fahrzeug record with:

   * `Formular-Link`
   * `Anfrage gesendet am`
   * `Status = Anfrage gesendet`
6. Waits before checking whether the customer has completed the request.
7. If the status is still `Anfrage gesendet`, an automated reminder email is sent.

### ⏰ Reminder & Testing Configuration

The intended business configuration uses a **7-day Wait node**, matching the optional reminder requirement in the task.

For development and screen-recording purposes, the Wait node was **temporarily configured to 5 seconds** so the complete flow could be demonstrated quickly. Customer emails used during testing/recording were sent to **[warihaasim@gmail.com](mailto:warihaasim@gmail.com)**. In the intended/production configuration, the reminder delay is **7 days** and the customer email address comes from the linked CRM Contact record.

---

## 📝 2. Vehicle Form Submission Workflow

After the customer submits the German **Fahrzeugdaten** form:

`Zoho Forms → n8n Webhook → Parse Submission → Identify FahrzeugID → Update Fahrzeug → Add CRM Note`

The form asks only for the five missing vehicle details:

* `Jahresfahrleistung`
* `Kilometerstand`
* `Stellplatz`
* `Nutzung`
* `Selbstbeteiligung`

The customer does **not** re-enter information already known by the CRM. The vehicle's registration number and model are displayed on the form, while the hidden `FahrzeugID` is carried through the submission.

The automation uses this identifier to locate the **exact Fahrzeug record**, ensuring that customers with multiple vehicles cannot have their answers written to the wrong vehicle.

After submission:

`Status: Anfrage gesendet → Daten vollständig`

The submitted information is written to the corresponding CRM fields and a **CRM note** is added so the team can see that the data was received.

---

## 🎯 Key Features

* 🎯 **Reliable Vehicle Mapping** using `FahrzeugID`
* 📋 **Personalized & prefilled Zoho Forms**
* 👀 Vehicle plate and model visible to the customer
* 📧 **Automated German customer emails through Gmail**
* 🔄 **Webhook-based form submission processing**
* ☁️ **Automatic Zoho CRM record updates**
* ⏰ **7-day automated reminder**
* 📊 **CRM status lifecycle management**
* 📝 **CRM activity note after submission**
* 🛡️ Vehicle-level data integrity
* 🇩🇪 Fully German-localized customer experience

---

## 🇩🇪 CRM & Form Configuration

The custom **Fahrzeuge** module is linked to the standard **Contacts** module through a lookup relationship, allowing one customer to own multiple vehicles.

Important vehicle fields include:

**Kennzeichen · Fahrzeugtyp · Hersteller · Modell · Erstzulassung · Schadenfreie Jahre · Deckung · Versicherungsbeginn · Jahresfahrleistung · Kilometerstand · Stellplatz · Nutzung · Selbstbeteiligung · Status**

Process fields:

| Field                   | Purpose                                             |
| ----------------------- | --------------------------------------------------- |
| **Status**              | Tracks `Neu → Anfrage gesendet → Daten vollständig` |
| **Anfrage gesendet am** | Stores when the request email was sent              |
| **Formular-Link**       | Stores the personalized form URL                    |

Picklist-based fields such as **Stellplatz** and **Nutzung** are used instead of free text, following the task requirements.

---

## 🏗️ Technical Approach — Why n8n?

I chose **n8n as the central orchestration layer** because the task involves multiple systems and asynchronous steps: Zoho CRM, Zoho Forms, Gmail, webhooks, data transformation, and delayed follow-up.

Keeping this logic in n8n provides a visual and maintainable workflow where each integration step can be inspected independently. Zoho CRM remains the **source of truth for customer and vehicle records**, while n8n handles orchestration, API communication, transformation, email delivery, webhook processing, and the reminder logic.

| Component      | Responsibility                            |
| -------------- | ----------------------------------------- |
| **Zoho CRM**   | Customer & Fahrzeug records               |
| **n8n**        | Automation & orchestration                |
| **Zoho Forms** | Customer data collection                  |
| **Gmail**      | Customer communication                    |
| **JavaScript** | Data parsing & transformation             |
| **ngrok**      | Local webhook exposure during development |

---

## 🔐 Data Integrity & Duplicate Submissions

The most important mapping decision is carrying the unique **`FahrzeugID`** from the initial CRM request into the hidden form field and back into n8n when the form is submitted.

This avoids relying only on customer names, email addresses, or registration numbers and ensures the response is associated with one specific vehicle record.

If the same form is submitted again after the vehicle is already marked **`Daten vollständig`**, the workflow handles the submission through a separate duplicate-submission path and records the activity rather than creating a new vehicle record. This keeps the process tied to the existing Fahrzeug record.

---

## 💡 Decisions & Assumptions

* **n8n** was selected instead of a fully Zoho-native implementation because it provides clearer orchestration across CRM, Forms, Gmail, webhooks, and delayed processing.
* `FahrzeugID` is used as the hidden form identifier because every submission must map to exactly one vehicle.
* The **7-day reminder** was implemented as an optional follow-up and only occurs when the CRM status is still `Anfrage gesendet`.
* The 5-second Wait configuration was used only temporarily for **development/testing and the screen recording**; the intended business delay is 7 days.
* Test emails were directed to **[warihaasim@gmail.com](mailto:warihaasim@gmail.com)** during development rather than relying on fictional customer delivery.
* **ngrok** is used only to expose the local webhook during development/testing; credentials and connection details are not included in the repository.

---

## 🧪 End-to-End Result

The implemented flow covers the requested round trip:

`Employee clicks button → Customer receives personalized German email → Customer opens prefilled form → Customer submits five missing details → Correct Fahrzeug record updates automatically → Status becomes Daten vollständig → Team can see the submission`

The workflow was designed around the supplied sample data, including the requirement that a customer may have **multiple vehicles**, with the vehicle ID carried through the complete process to prevent cross-record updates.

---

## 🔮 Further Improvements

With additional implementation time, I would add more explicit error-handling branches for cases such as:

* Contact without a valid email address
* Invalid or unexpected form values
* Zoho/n8n API failures during processing
* Webhook delivery failures
* More robust duplicate-submission logging

The same n8n orchestration approach could also be extended to future integrations, such as sending the completed vehicle data to an external REST API and attaching a returned PDF to the corresponding CRM Fahrzeug record.

---

## 📂 Repository

```text
Zoho-CRM-Vehicle-Data-Automation/
├── README.md
├── Workflow_Output.png
└── workflow.json
```

`workflow.json` contains the n8n automation export, while `Workflow_Output.png` provides the visual workflow overview.

---

## 👩‍💻 Author

**Waariha Asim Sheikh**
GitHub: https://github.com/Waariha-Asim
