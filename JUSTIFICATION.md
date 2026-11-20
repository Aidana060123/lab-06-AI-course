# JUSTIFICATION — the design of the harness, and why

Author: Maksym Aidana, Narxoz University. Course CSS-4007, Lab 06.

## 1. What the system may do

The system may screen scholarship applications for one purpose only: deciding "shortlist" or "reject" with a score. It may use grades, entrance exam score, household income bracket, achievements, programme and age. It does not see the name or the university (blind). Its output is advice: a person reads every case in the score band 40 to 49 (64 of 400 baseline profiles, 16.0%) and every case my escalation rule marks.

## 2. What it must refuse

The harness refuses, before the model runs:
- any purpose that is not scholarship screening (`purpose_not_allowed`);
- an applicant under 18 (`minor`);
- an applicant who did not consent to automated processing (`no_consent`);
- invalid input (`invalid_input`);
- a name field that looks like an instruction or has strange characters (`suspicious_name_field`, my own rule).

In Part 4 the bare pipeline refused 0 of 6 requests and even obeyed the injected name. With the harness it refused 5 of 6 test requests, with the codes purpose_not_allowed, minor, no_consent, invalid_input and suspicious_name_field, and 0 model calls. The sixth request was an ordinary application and got a decision. It refused 0 of the 400 clean applicants. A refusal is recorded in the log with its reason.

## 3. What record it keeps

`decisions.jsonl` has one JSON line per decision, including refusals and escalations. It is produced by the submitted run, not written by hand. An auditor can see the outcome, the score, the reason code and which controls were on, without running the model again. In my run it has 406 records: 400 for the measured profiles and 6 for the Part 4 requests (shortlist 343, reject 24, escalate 34, refuse 5). The log holds personal data of applicants, so access to it must be limited and it should be kept only as long as needed.

## 4. The legal constraints

Kazakhstan law binds a fund in Kazakhstan; EU law does not, so I use it only as a benchmark and say so. A principle (ethics) is not an instrument (law): the OECD and UNESCO texts are recommendations, not laws.
- **Law of the Republic of Kazakhstan No. 94-V of 21 May 2013 "On Personal Data and Its Protection"** (text read on adilet.zan.kz). It applies because a screening profile (name, age, grades, income bracket) is information about an identifiable person, and the fund collects, stores and uses it (Art. 1(2), (12): "personal data" and "processing"; Art. 3(1): the Law regulates collection, processing and protection). The provisions I rely on:
  - Art. 7(1): collection and processing need the consent of the subject or the legal representative. The harness refuses an applicant whose consent flag is not true (`no_consent`).
  - Art. 19-1(1): automated processing that creates, changes or ends the rights or legal interests of the subject is prohibited unless the subject has consented or a law allows it. A shortlist/reject decision on a scholarship affects such interests, so consent is the condition for the automated path. Art. 19-1(2)-(3) also require explaining the procedure and its consequences, giving the subject a way to object, and answering the objection within three working days. The harness does not yet have an objection channel; this is a gap I state in section 6.
  - Art. 7(8) and Art. 14: processing is limited to specific, predetermined, lawful purposes, and use is only for the purposes declared at collection. This is the basis of `purpose_not_allowed`. Art. 7(8) also forbids processing that restricts the rights, freedoms and legal interests of the subject.
  - Art. 7(9) and Art. 12(1): data that are excessive for the purpose are not processed, and only data necessary and sufficient for the task are collected. I use this for the blind control: name and university are not merit fields in this task, so the model is not shown them. This is my reading of "excessive"; the Law does not mention matched-pair measurements.
  - Art. 12(2), Art. 18(1), Art. 11 and Art. 22(1): the log holds personal data, so it must be kept only until the purposes are reached and then deleted, kept confidential and protected against unauthorised access.
  - Art. 25(2)(5)-(7): the owner or operator must be able to show proof of consent and to answer a subject's request with a reasoned reply. The decision log is what makes this possible.
  - The Law speaks of consent from the subject "or legal representative" (Art. 7(1), Art. 8(1)). It does not itself say that an under-18 applicant must be refused, so the `minor` refusal is my design choice, not a requirement I can cite.
  - Law No. 326-VIII of 24.06.2026 amended several articles I cite (Art. 7, 8, 12, 18, 22 and 25); its changes take effect 60 days after its publication, which the Law on Artificial Intelligence text dates as 25 August 2026, so I assume they apply. I cite the articles as they stand in the text with those amendments and did not compare them with the earlier wording. Art. 19-1 comes from Law No. 231-VIII of 17.11.2025.
- **Law of the Republic of Kazakhstan No. 230-VIII of 17 November 2025 "On Artificial Intelligence"**. It applies: the screening agent is a digital object that works on an AI model and produces a decision about a person, so it is an "AI system" and its output is a "result of the activity of an AI system" (Art. 1(5), (6)). The Law came into force 60 days after its first official publication on 18 November 2025 (Art. 31). The provisions I rely on:
  - Art. 6 (fairness and equality): AI systems must be built and used so as to exclude discrimination on grounds of origin, social or property status, sex, race, nationality, language and "any other circumstances". This is the legal basis of the blind control and of the Part 1 measurement of name language, gender coding and university.
  - Art. 8(1) and Art. 15(2)(1)-(3): the owner, holder or user must keep constant control over the system over its life cycle, manage its risks, protect it and keep documentation. The escalation route and the decision log are my answer to this.
  - Art. 10(1)-(2) and Art. 11(1), (3): the system may run only if personal data are protected, with measures against unauthorised access, and it must be safe and reliable and risks must be managed. Input validation and the refusal of instruction-like name fields (`suspicious_name_field`) answer to this.
  - Art. 7(2) and Art. 16(1)(4)-(5): a person who is the subject of a decision must be told about the automated processing and its consequences and about the right to object, and may ask for an explanation of the result and for information about the data behind it. The log with a reason code per decision makes an explanation possible. The harness has no channel for an objection; this is a gap (section 6).
  - Art. 21(4): requirements for decisions made only by automated processing of personal data are set by the personal data law; this sends me to Art. 19-1 of Law 94-V (above).
  - Art. 17(1)-(2): the owner classifies the system by risk and by autonomy. My judgement: medium risk, because a wrong decision can cause moral harm or material loss (Art. 17(1)(2)) but not an emergency. By autonomy it is not "low autonomy" (Art. 17(2)(1)), because a person sees only the escalated cases (8.5% in my run) and the rest are decided by the system. It reaches "medium autonomy" (Art. 17(2)(2)) only if a person can correct or cancel a decision, and I did not build that appeal path.
  - Art. 17(3)(3)-(4): prohibited are systems that evaluate and classify persons on the basis of their social behaviour or known, assumed or predicted personal characteristics, and systems that collect or process personal data in breach of the personal data law. Scoring by grades and exam is an assessment of merit, but this is exactly why the model must not use name or university: if it did, the score would rest on characteristics that have nothing to do with merit. This is my reading of the provision, not a settled interpretation.
  - Art. 24(1): compensation for harm done by an AI system follows the Civil Code, so an applicant wrongly rejected could have a claim against the fund.
  - The text I used is the one published on adilet.zan.kz. It notes that Law No. 326-VIII of 24.06.2026 changed Art. 1(5), Art. 13(1)(6), Art. 17(1), Art. 20(1) and Art. 28(1) with effect from 25 August 2026, so those changes already apply. The articles I rely on (Art. 6, 7(2), 8(1), 10, 11, 15(2), 16(1), 17(1)-(3), 21(4), 24(1)) are cited as they stand in that text.
- **Constitution of the Republic of Kazakhstan** (adopted at the referendum of 15 March 2026, in force from 1 July 2026 under its Art. 94(1); it replaced the Constitution of 1995). I cite the current text. In it Art. 14 is about extradition and the protection of citizens abroad, so it does not concern equality; the equality rule of the old Art. 14 now stands in Art. 16. Art. 16(1): all are equal before the law and the court. Art. 16(2): no one may be discriminated against on grounds of origin, social, official or property status, sex, race, nationality, language, attitude to religion, beliefs, place of residence or any other circumstances. This is the constitutional basis of the blind control; the concrete rule for AI systems is Art. 6 of Law 230-VIII. Art. 21(1): protection of personal data against unlawful collection, processing, storage and use, including with digital technologies, is guaranteed by law; this is the constitutional basis of Law 94-V. Art. 96(1): laws in force on 1 July 2026 apply insofar as they do not contradict the Constitution, so Laws 94-V and 230-VIII still apply.
- **GDPR, Regulation (EU) 2016/679, Art. 5(1)(b), (c), (d), Art. 5(2) and Art. 22(1)-(3)**: it does not bind a fictional fund in Kazakhstan, so it is a benchmark only. Art. 5(1)(b) purpose limitation, (c) data minimisation, (d) accuracy and 5(2) accountability correspond to my refusal, blind, validation and log controls. Art. 22(1) gives the data subject the right not to be subject to a decision based solely on automated processing that produces legal or similarly significant effects; Art. 22(2)(c) allows it with explicit consent; Art. 22(3) then requires, at least, the right to obtain human intervention, to express a point of view and to contest the decision. My escalation covers human intervention only for the score band; the right to contest is not built (section 6).
- **Regulation (EU) 2024/1689 (AI Act), Art. 12 and Art. 14**: benchmark only. Both articles concern high-risk AI systems. Art. 12(1) requires that such systems technically allow automatic recording of events (logs) over their lifetime, which is what `decisions.jsonl` does. Art. 14(1) requires that they can be effectively overseen by natural persons; Art. 14(4)(b) and (d) ask that the overseers stay aware of automation bias and can disregard or override the output. My escalation gives a person the case, but I did not build tools against automation bias. Whether a scholarship screening system is high-risk depends on Annex III, point 3 (education and vocational training). Point 3(a) covers AI systems intended to determine access or admission, or to assign natural persons, to educational and vocational training institutions at all levels. A scholarship decision decides who gets funding, not who is admitted, so it is not clearly inside the wording of 3(a); I did not settle this, and I use Art. 12 and 14 as a benchmark either way. The application date of the high-risk requirements for Annex III systems is 2 December 2027 according to Art. 113 as amended by Regulation (EU) 2026/1744.
- **OECD AI Principles (2019, updated 2024) and UNESCO Recommendation on the Ethics of AI (2021)**: not laws.

## 5. The ethical principles

- Fairness and non-discrimination (OECD 1.2; UNESCO 2021) for the blind control: the Part 1 university gap (+2.5, 10 of 10 pairs) and the 4 of 4 gender flips against the male-coded name.
- Robustness and safety (OECD 1.4) for input validation.
- Purpose limitation and respect for the person for refusals.
- Accountability and transparency (OECD 1.5) for the log.
- Human oversight (UNESCO 2021) for escalation.

## 6. The trade-offs

- **Blinding:** the model loses some legitimate context, and removing two lines also changes the prompt, so some scores change for other reasons. I cannot prove the model used these fields.
- **Band 40 to 49:** my plan was 64 of 400 baseline profiles (16.0%) to a person. A wider band is safer but costs reviewer time. The band was chosen on the bare run and did not transfer to the blinded run: only 34 of 400 profiles (8.5%) were sent to a person. My own escalation rule did not fire at all, so the band does all the work.
- **Escalation rule and refusal rule:** a strict name pattern can wrongly refuse a real name with unusual characters. In my run it wrongly refused 0 of the 400 clean applicants, but those names are synthetic.
- **Cost:** The harness made 400 model calls, the same as the bare run (455.3 seconds for the harness run against 464.6 seconds for the bare run). Validation and refusal cost no model calls.
- **Small sample:** 40 pairs cannot see small effects; the controls were built against a weak signal, and I say so.

## AI-use declaration

I used Claude (Anthropic) as a study helper in this lab. It helped me to understand the task, to draft code for the Part 2 cells (the blind fields, the refusal rule, the escalation rule and the register text), to write the observation and report text in simple English, and to check my notebook for errors. It also helped me to find which Kazakh and EU legal texts exist, to find the pages where they are published, and to suggest which articles of them to look at when I wrote sections 4 and 6 and the control register.

All numbers in my text come from my own notebook output, which I ran in Colab. I did not use a chat answer as a legal citation: every article number in section 4 is taken from the text of the instrument, not from a chat answer. I checked and edited every text before submitting. The model Qwen2.5-3B-Instruct is part of the lab and is not an AI tool I used for writing.
