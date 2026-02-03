
  1. What is the single most valuable outcome for doctors in the first 3 months of use?
    
	 So, doctors in India are mostly tied up to the tasks which are basically non-productive in nature. Some may or may not be productive in nature.  Most of us are getting messages on our WhatsApp. Some of these are appointment requests. Some of these are patients sending their PDFs and images of their reports asking our opinion. Most of us get the consultation requests to be done immediately. Most of us are alarmed about some ECG. We never know that our WhatsApp has some incoming ECG, some PDFs or other things. Triaging whatsapp is important, and in bigger hospitals, people have their secretaries who are between this doctor and patient layer. Patients usually send their reports and their ECGs and PDFs and appointment requests to the secretary. And such secretaries make the life of doctors easier. They find out: Which of the patient is asking for the appointment - Which is requesting for the reports to be seen. Which is having some medical emergency and needs to be called to the hospital immediately. Appointment coordination is made very easy if there is an available secretary with the doctors.
	 I So what I want to see is every medical practitioner even in living in Tier 3 city of India having the capabilities of a staff that doctor in a corporate hospital in a Tier 1 city like Delhi and Mumbai has. And yes, I know that that there are apps available for almost everything that I've talked about. As doctors, we do not want to be fighting 10 different tools for this job. This is just wasting time thinking that you are saving time, and it does not frankly translate into increased productivity using all those tools. 
	 
  1. What would make a doctor say “this is better than a human secretary”?
I have seen these secretaries work. They ask for holidays, they do not pick up phones for coordinating appointments when the duty time is off. They need a periodic raise in their salaries, and not everybody can afford a human help. It is not going to come free of cost, and it is a high pressure job. Most of the times, there can be conflict between our doctor and their secretaries because of some issues. The secretary might not take call when it is important, and they might not realize that what they did. Something which increases productivity, saves my time, saves my money, is continuously available, and takes care of a cognitive load would definitely be better than a secretary. 
 
  2. What should the product not do (explicit exclusions)?
   no doctor would like the secretary to auto prescribe on his or her behalf. That is for sure. I would not like my secretary replying to my patients on WhatsApp that you should be taking X and Y drug and come back to us with X and Y reports after four, five days without taking my approval. This is not... This is... Product should not do this. It should find out which of the patients who are asking for such help and I will write the prescription, maybe I'll guide the product to do this but it should not be auto prescribing and that is most important thing. Whether the product is living in EMR app or patient's WhatsApp and my WhatsApp, it should not be auto prescribing

  Users & Settings
  4. Which specialties are your first targets?
   I am a cardiologist, so my priority would be showing this to cardiologists first and general practitioners also because they would be easy to find and easy to target 
  5. Typical practice setting: solo clinic, multi‑doctor clinic, or hospital department?
  big hospitals already have secretaries available to the doctors, and my target is going for the solo clinics and individual doctors primarily in my initial setups. And maybe when I expand to a larger setup, then I'll think of how can we expand, how can we make the software better. Maybe we should be giving some hardware with it, or attaching some more features, bills, and missiles. Maybe we should be adding some capabilities of inter-departmental talk between the prescriptions. Those are the things I'll consider later on, but currently I'm targeting individual doctors. I'm thinking of increasing their productivity and decreasing their pains 
  6. How many patients/messages per doctor per day in your target setting?
   honestly, I did not get this question. In usual practice, what happens is none of the requests coming from the patients are declined by any kind of doctor 
  7. Are doctors or existing secretaries the primary users?
 primary users are the doctors only, not their secretaries. The aim is replacing secretaries 
  Workflow & Use Cases
  8. What are the top 5 message types you want the assistant to handle?
appointment request 
opinions on reports. Reports may be incoming in the form of PDFs, JPEGs, or PNGs 
OPD availability on the day.  the secretary usually tells the patients that Doxab is available on this particular day or not. So that is one of the requests that we usually get, and that is one of the requests that I want to be taken care of. 
some of the patients do not understand what is OD and BD, so at times that needs to be explained - although that would be simplified by the app that we are building. That would also be a prescription writer that would categorically tell the patients that OD and BD stand for. But sometimes patients do ask silly questions, which of the following medications are supposed to be taken in the morning, which of the following medications are taken to be in the evening. So that is one aspect that I would like to be taken care of. 
 then patients usually ask for alternative medications. Some of the times medications are not available in the pharmacy that they are looking for. So they ask for the alternative brand names that they want to use. Alternative brands, this is one thing that people like to know, and this is one of the common requests that we get in our messages 
 then there are messages in which patients usually ask, "With what kind of investigation should I come up with during my follow-up?" Those are some specialty-specific investigations. In my specialty, there is a certain number of tests that I usually order most of the post-PTCA patients, post-CABG patients. So, in every specialty, there is a set number of tests which usually patients are asked to get. It is not a pattern, but most of the tests make sense in majority of these people 
 
  9. What are the top 3 “urgent” cases the assistant must escalate immediately?
   medical emergencies have to be identified by all means and they should be escalated to the doctor immediately. Attention should be brought to both the doctor that some action is urgently required from your side, and also the patient should also be made aware that this is important 
  10. What is the ideal triage outcome set (e.g., respond, schedule, escalate, request info)?
  yeah, that makes sense. The response schedule escalates the request info that makes sense 
  11. How should the system handle after‑hours messages?
  so, that depends upon different specialties and different types of complaints which are incoming. Most of the times, when there is a routine request, I would like it to respond like a human would. Not to reply after office hours. Some doctors would like that after office hours also. Such a message as an interring. But all doctors would like that emergencies have to be correctly identified and escalated.

  Communication Channels
  12. Which channels are mandatory at launch (WhatsApp, SMS, phone, email, in‑app)?
  as far as I understand, nobody is going to install a separate app for this purpose. No patient is going to install a separate, untested app for merely talking to the doctor. I don't know of people who would gladly download apps that make no sense outside of consultations. I myself have encountered multiple apps which came and went by and nobody downloaded them. Most people would instantly delete them after talking to the doctor, so it would make sense if the 
  13. Are patients already using a specific channel that cannot change?
  14. Do you need voice calls or only text/chat?

  Prescriptions & Documentation
  15. What is the minimum required prescription output format (PDF, print, WhatsApp image, etc.)?
  16. Are prescriptions primarily handwritten today or typed?
  17. Do you want OCR from handwritten notes in v1, or manual entry assisted by AI?

  Data & Records
  18. What patient data is essential for v1 (name, age, diagnosis, meds, allergies, etc.)?
  19. Do you need full EMR storage or only a “conversation + summary” record?
  20. How long should data be retained?

  Compliance & Safety
  21. Which Indian regulations or standards must be followed?
  22. What is your risk tolerance for AI errors (low/medium/high)?
  23. Should the assistant ever send medical advice directly to patients without doctor review?

  Integrations
  24. Do you need integrations with existing EMRs (HealthPlex, Practo) or start standalone?
  25. Which external services are required (SMS gateway, WhatsApp Business API, payment, labs)?
  26. Are there existing hospital systems you must connect to?

  AI Behavior & Controls
  27. How should the assistant explain uncertainty to patients?
  28. What should be the escalation thresholds for humans?
  29. Do you want templated responses or free‑form AI replies?

  Deployment & Operations
  30. Cloud vs on‑premise vs hybrid?
  31. Do you need offline or low‑bandwidth support?
  32. Who will maintain the system (your team vs clinic IT)?

  Business & Go‑to‑Market
  33. Pricing model preference (per doctor, per clinic, per message)?
  34. Who signs the contract and who pays?
  35. What is the expected onboarding effort per clinic?

  Timeline & Resources
  36. What is your desired timeline for MVP?
  37. How many engineers/designers are available?
  38. What is your budget range for initial development and infrastructure?

  Metrics
  39. What metrics define success in the pilot (time saved, response time, patient satisfaction)?
  40. What is an acceptable error rate for triage/response?
