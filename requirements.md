# Requirements Document: GramaCare AI Healthcare Platform

## Introduction

GramaCare AI is an AWS-native serverless healthcare platform designed to provide AI-powered health triage, risk assessment, and care navigation for rural communities in India. The platform addresses the critical healthcare access gap in underserved areas by offering multi-channel access (web, mobile apps, SMS) with intelligent symptom assessment, risk classification, and appropriate care routing.

The system leverages AWS cloud services to deliver scalable, secure, and compliant healthcare services while maintaining high availability and performance standards suitable for rural connectivity conditions.

## Glossary

- **Platform**: The GramaCare AI healthcare system
- **User**: A rural patient or community member accessing healthcare services
- **ASHA_Worker**: Accredited Social Health Activist, a community health facilitator
- **Healthcare_Provider**: Medical professional providing teleconsultation or care guidance
- **System_Admin**: Administrator managing platform configuration and operations
- **Triage_Service**: AI-powered service that assesses symptoms and classifies risk
- **Risk_Level**: Classification of health urgency (Low, Moderate, High)
- **Care_Path**: Recommended course of action based on risk assessment
- **OTC**: Over-the-counter medication or self-care guidance
- **Symptom_Data**: Structured information about patient symptoms and medical history
- **Medical_Document**: Patient-uploaded photos or documents related to health condition
- **Consent_Record**: User authorization for data collection and processing
- **Authentication_Service**: AWS Cognito-based identity management with OTP
- **ML_Model**: Machine learning model for risk classification hosted on SageMaker
- **LLM_Service**: Large Language Model service via Amazon Bedrock for explanations
- **Notification_Service**: System for sending SMS, push, and email notifications
- **Health_Record**: Historical patient data including symptoms, assessments, and outcomes
- **Teleconsultation**: Remote video/audio consultation with healthcare provider
- **Medical_Summary**: Generated PDF report of patient health assessment and history
- **DPDP_Act**: Digital Personal Data Protection Act 2023 (India)
- **Encryption_Service**: AWS KMS-based encryption for data at rest and in transit

## Requirements

### Requirement 1: User Registration and Consent Management

**User Story:** As a rural user, I want to register for the platform and provide informed consent, so that I can access healthcare services while understanding how my data will be used.

#### Acceptance Criteria

1. WHEN a new user accesses the registration interface, THE Platform SHALL display registration options for phone number or email
2. WHEN a user submits valid registration information, THE Platform SHALL create a user account in the Authentication_Service
3. WHEN creating an account, THE Platform SHALL present consent information in the user's preferred language
4. WHEN a user provides consent, THE Platform SHALL store the Consent_Record with timestamp and consent version
5. WHEN a user completes registration, THE Platform SHALL send a verification code via SMS or email within 30 seconds
6. IF a user attempts to register with an already-registered phone number or email, THEN THE Platform SHALL return an appropriate error message
7. WHEN a user views their consent history, THE Platform SHALL display all Consent_Records with dates and versions
8. WHEN a user withdraws consent, THE Platform SHALL mark the Consent_Record as withdrawn and restrict data processing

### Requirement 2: Authentication and Login

**User Story:** As a user, I want to securely log in using OTP, so that I can access my health information without remembering complex passwords.

#### Acceptance Criteria

1. WHEN a user initiates login, THE Platform SHALL request phone number or email identifier
2. WHEN a valid identifier is provided, THE Authentication_Service SHALL generate a 6-digit OTP
3. WHEN an OTP is generated, THE Platform SHALL send it via SMS or email within 30 seconds
4. WHEN a user submits a valid OTP within 10 minutes, THE Authentication_Service SHALL create an authenticated session
5. WHEN a user submits an invalid OTP, THE Platform SHALL increment the failed attempt counter
6. IF a user exceeds 3 failed OTP attempts, THEN THE Platform SHALL temporarily lock the account for 15 minutes
7. WHEN a user's session expires after 24 hours, THE Platform SHALL require re-authentication
8. WHEN a user logs out, THE Platform SHALL invalidate the current session token immediately

### Requirement 3: Structured Symptom Intake

**User Story:** As a user, I want to describe my symptoms through a guided questionnaire, so that the system can accurately assess my health condition.

#### Acceptance Criteria

1. WHEN a user starts symptom intake, THE Platform SHALL present a structured questionnaire in the user's preferred language
2. WHEN a user selects a primary symptom, THE Platform SHALL display relevant follow-up questions based on medical protocols
3. WHEN a user provides symptom information, THE Platform SHALL validate input for completeness and medical relevance
4. WHEN a user uploads a Medical_Document, THE Platform SHALL accept image formats (JPEG, PNG) up to 10MB
5. WHEN a Medical_Document is uploaded, THE Platform SHALL store it in encrypted S3 storage within 5 seconds
6. WHEN a user completes the questionnaire, THE Platform SHALL save the Symptom_Data to DynamoDB with timestamp
7. IF a user provides incomplete critical information, THEN THE Platform SHALL prompt for required details before proceeding
8. WHEN symptom intake is saved, THE Platform SHALL generate a unique assessment ID for tracking

### Requirement 4: ML-Based Risk Classification

**User Story:** As a user, I want my symptoms to be automatically assessed for urgency, so that I receive appropriate care recommendations quickly.

#### Acceptance Criteria

1. WHEN Symptom_Data is submitted, THE Triage_Service SHALL invoke the ML_Model for risk classification
2. WHEN the ML_Model processes symptoms, THE Platform SHALL return a Risk_Level within 2 seconds
3. WHEN a Risk_Level is determined, THE ML_Model SHALL provide a confidence score between 0 and 1
4. WHEN the confidence score is below 0.7, THE Platform SHALL flag the assessment for Healthcare_Provider review
5. WHEN risk classification completes, THE Platform SHALL store the Risk_Level and confidence score with the assessment
6. WHEN a High Risk_Level is assigned, THE Platform SHALL immediately trigger emergency notification workflows
7. WHEN the LLM_Service generates an explanation, THE Platform SHALL provide it in the user's preferred language
8. IF the ML_Model fails to respond within 3 seconds, THEN THE Platform SHALL default to Moderate risk and log the failure

### Requirement 5: Care Path Routing

**User Story:** As a user, I want to receive appropriate care recommendations based on my risk level, so that I get the right level of care for my condition.

#### Acceptance Criteria

1. WHEN a Low Risk_Level is assigned, THE Platform SHALL provide OTC guidance and self-care instructions
2. WHEN a Moderate Risk_Level is assigned, THE Platform SHALL recommend monitoring and schedule a follow-up check
3. WHEN a High Risk_Level is assigned, THE Platform SHALL immediately display emergency contact information
4. WHEN a High Risk_Level is assigned, THE Platform SHALL send automated alerts to registered emergency contacts
5. WHEN a Care_Path is determined, THE Platform SHALL display next steps in clear, actionable language
6. WHEN OTC guidance is provided, THE Platform SHALL include medication names, dosages, and precautions
7. WHEN emergency escalation occurs, THE Platform SHALL log the escalation event with timestamp for audit
8. WHEN a user views their Care_Path, THE Platform SHALL display the reasoning based on their symptoms

### Requirement 6: Medical Photo Upload and Storage

**User Story:** As a user, I want to upload photos of my condition, so that healthcare providers can better assess my situation.

#### Acceptance Criteria

1. WHEN a user selects a photo to upload, THE Platform SHALL validate the file type and size before upload
2. WHEN a valid photo is selected, THE Platform SHALL compress images larger than 5MB while maintaining diagnostic quality
3. WHEN a photo upload begins, THE Platform SHALL display upload progress to the user
4. WHEN a photo is uploaded, THE Encryption_Service SHALL encrypt it before storing in S3
5. WHEN a photo is stored, THE Platform SHALL associate it with the user's assessment ID
6. WHEN a Healthcare_Provider views an assessment, THE Platform SHALL decrypt and display associated Medical_Documents
7. IF a photo upload fails, THEN THE Platform SHALL allow retry up to 3 attempts
8. WHEN a user deletes a photo, THE Platform SHALL remove it from S3 and update the assessment record

### Requirement 7: Preventive Health Recommendations

**User Story:** As a user, I want to receive personalized preventive health advice, so that I can maintain good health and prevent future issues.

#### Acceptance Criteria

1. WHEN a user completes an assessment, THE Platform SHALL analyse Health_Record for preventive opportunities
2. WHEN preventive recommendations are generated, THE Platform SHALL personalize them based on age, gender, and history
3. WHEN recommendations are displayed, THE Platform SHALL present them in simple, culturally appropriate language
4. WHEN a user views recommendations, THE Platform SHALL include actionable steps and timelines
5. WHEN seasonal health risks are identified, THE Platform SHALL proactively send preventive guidance
6. WHEN a user acknowledges a recommendation, THE Platform SHALL track engagement for outcome analysis
7. WHEN recommendations include vaccinations, THE Platform SHALL provide information on local availability
8. WHEN a user requests more information, THE LLM_Service SHALL provide detailed explanations

### Requirement 8: Medication Reminders

**User Story:** As a user, I want to receive reminders for my medications, so that I can follow my treatment plan consistently.

#### Acceptance Criteria

1. WHEN a user sets up a medication reminder, THE Platform SHALL accept medication name, dosage, and schedule
2. WHEN a reminder schedule is created, THE Platform SHALL validate the timing and frequency
3. WHEN a reminder time arrives, THE Notification_Service SHALL send an SMS or push notification
4. WHEN a user confirms taking medication, THE Platform SHALL log the adherence event with timestamp
5. WHEN a user misses a medication, THE Platform SHALL send a follow-up reminder after 30 minutes
6. WHEN a user views medication history, THE Platform SHALL display adherence rates and missed doses
7. WHEN a medication course completes, THE Platform SHALL notify the user and archive the reminder
8. WHEN a user modifies a reminder, THE Platform SHALL update the schedule and confirm the changes

### Requirement 9: Follow-up Scheduling

**User Story:** As a user, I want to schedule follow-up check-ins, so that I can monitor my condition over time.

#### Acceptance Criteria

1. WHEN a Care_Path recommends follow-up, THE Platform SHALL suggest appropriate follow-up intervals
2. WHEN a user schedules a follow-up, THE Platform SHALL create an event in the scheduling system
3. WHEN a follow-up is scheduled, THE Platform SHALL send confirmation via SMS or push notification
4. WHEN a follow-up time approaches, THE Platform SHALL send a reminder 24 hours in advance
5. WHEN a follow-up is due, THE Platform SHALL prompt the user to complete a new symptom assessment
6. WHEN a user completes a follow-up assessment, THE Platform SHALL compare it with the previous assessment
7. WHEN condition improvement is detected, THE Platform SHALL provide positive reinforcement
8. IF condition worsening is detected, THEN THE Platform SHALL escalate to a Healthcare_Provider

### Requirement 10: Teleconsultation Initiation

**User Story:** As a user, I want to request a teleconsultation with a healthcare provider, so that I can receive professional medical advice remotely.

#### Acceptance Criteria

1. WHEN a user requests teleconsultation, THE Platform SHALL display available Healthcare_Provider slots
2. WHEN a user selects a time slot, THE Platform SHALL reserve it for 10 minutes pending confirmation
3. WHEN a teleconsultation is booked, THE Platform SHALL send confirmation to both user and Healthcare_Provider
4. WHEN a teleconsultation time approaches, THE Platform SHALL send reminders to both parties 15 minutes before
5. WHEN a teleconsultation starts, THE Platform SHALL provide a secure video/audio connection
6. WHEN a teleconsultation is in progress, THE Platform SHALL display the user's Health_Record to the Healthcare_Provider
7. WHEN a teleconsultation ends, THE Platform SHALL prompt the Healthcare_Provider to add consultation notes
8. WHEN consultation notes are saved, THE Platform SHALL update the user's Health_Record immediately

### Requirement 11: Medical Summary Generation

**User Story:** As a user, I want to download a summary of my health assessments, so that I can share it with healthcare providers or keep for my records.

#### Acceptance Criteria

1. WHEN a user requests a Medical_Summary, THE Platform SHALL compile all relevant Health_Record data
2. WHEN generating a summary, THE Platform SHALL include assessments, risk classifications, and care paths
3. WHEN a Medical_Summary is created, THE Platform SHALL format it as a PDF document
4. WHEN a PDF is generated, THE Platform SHALL include patient demographics, assessment dates, and outcomes
5. WHEN a Medical_Summary is ready, THE Platform SHALL provide a download link valid for 7 days
6. WHEN a user downloads a summary, THE Platform SHALL log the access event for audit purposes
7. WHEN a summary includes Medical_Documents, THE Platform SHALL embed them in the PDF
8. WHEN a user shares a summary, THE Platform SHALL generate a secure, time-limited sharing link

### Requirement 12: Health History and Records Viewing

**User Story:** As a user, I want to view my complete health history, so that I can track my health journey and share information with providers.

#### Acceptance Criteria

1. WHEN a user accesses health history, THE Platform SHALL display all past assessments in reverse chronological order
2. WHEN displaying history, THE Platform SHALL show assessment date, symptoms, Risk_Level, and Care_Path
3. WHEN a user selects a specific assessment, THE Platform SHALL display full details including recommendations
4. WHEN viewing history, THE Platform SHALL provide filtering options by date range and Risk_Level
5. WHEN a user searches history, THE Platform SHALL return matching assessments within 1 second
6. WHEN displaying Medical_Documents, THE Platform SHALL show thumbnails with option to view full size
7. WHEN a user exports history, THE Platform SHALL generate a comprehensive Medical_Summary
8. WHEN history data is accessed, THE Platform SHALL log the access for DPDP_Act compliance

### Requirement 13: Patient Outcome Tracking

**User Story:** As a Healthcare_Provider, I want to track patient outcomes, so that I can evaluate treatment effectiveness and improve care quality.

#### Acceptance Criteria

1. WHEN a patient completes a follow-up assessment, THE Platform SHALL compare outcomes with initial assessment
2. WHEN tracking outcomes, THE Platform SHALL calculate improvement, stability, or deterioration metrics
3. WHEN outcomes are analyzed, THE Platform SHALL aggregate data while maintaining patient privacy
4. WHEN a Healthcare_Provider views outcomes, THE Platform SHALL display trends and patterns
5. WHEN outcome data is exported, THE Platform SHALL anonymize patient identifiers
6. WHEN negative outcomes are detected, THE Platform SHALL flag cases for provider review
7. WHEN outcome reports are generated, THE Platform SHALL include confidence intervals and sample sizes
8. WHEN a System_Admin requests analytics, THE Platform SHALL provide outcome dashboards with visualizations

### Requirement 14: System Administration

**User Story:** As a System_Admin, I want to manage platform configuration and monitor system health, so that I can ensure reliable service delivery.

#### Acceptance Criteria

1. WHEN a System_Admin logs in, THE Platform SHALL verify admin privileges via Authentication_Service
2. WHEN viewing system health, THE Platform SHALL display real-time metrics from CloudWatch
3. WHEN configuring ML_Model parameters, THE Platform SHALL validate changes before applying
4. WHEN updating consent forms, THE Platform SHALL version the changes and notify active users
5. WHEN managing user accounts, THE Platform SHALL provide search, view, and deactivation capabilities
6. WHEN reviewing audit logs, THE Platform SHALL display all access events with timestamps and user IDs
7. WHEN system errors occur, THE Platform SHALL send alerts to System_Admin via SNS
8. WHEN a System_Admin modifies configuration, THE Platform SHALL log the change with admin ID and timestamp

### Requirement 15: Analytics and Reporting

**User Story:** As a System_Admin, I want to generate analytics reports, so that I can understand platform usage and health trends in the community.

#### Acceptance Criteria

1. WHEN generating analytics, THE Platform SHALL aggregate data across all users while preserving privacy
2. WHEN displaying usage metrics, THE Platform SHALL show daily, weekly, and monthly active users
3. WHEN analyzing health trends, THE Platform SHALL identify common symptoms and Risk_Level distributions
4. WHEN creating reports, THE Platform SHALL provide export options in CSV and PDF formats
5. WHEN geographic analysis is requested, THE Platform SHALL display health trends by region
6. WHEN time-series analysis is performed, THE Platform SHALL identify seasonal patterns
7. WHEN demographic analysis is conducted, THE Platform SHALL segment by age groups and gender
8. WHEN reports are generated, THE Platform SHALL ensure compliance with DPDP_Act anonymization requirements

### Requirement 16: Data Privacy and Security

**User Story:** As a user, I want my health data to be protected and private, so that I can trust the platform with sensitive information.

#### Acceptance Criteria

1. WHEN data is stored, THE Encryption_Service SHALL encrypt it at rest using AES-256
2. WHEN data is transmitted, THE Platform SHALL use TLS 1.3 for encryption in transit
3. WHEN a user requests data deletion, THE Platform SHALL remove all personal data within 30 days
4. WHEN accessing sensitive data, THE Platform SHALL verify user authorization via Authentication_Service
5. WHEN data is shared with Healthcare_Provider, THE Platform SHALL log the access with purpose and timestamp
6. WHEN a data breach is detected, THE Platform SHALL notify affected users within 72 hours
7. WHEN audit logs are created, THE Platform SHALL retain them for 7 years per compliance requirements
8. WHEN a user exports their data, THE Platform SHALL provide it in a machine-readable format within 48 hours

### Requirement 17: Performance and Availability

**User Story:** As a user, I want the platform to be fast and always available, so that I can access healthcare services when I need them.

#### Acceptance Criteria

1. WHEN a user makes an API request, THE Platform SHALL respond within 500 milliseconds
2. WHEN the ML_Model performs inference, THE Platform SHALL return results within 2 seconds
3. WHEN the Platform experiences high load, THE Platform SHALL auto-scale Lambda functions to maintain performance
4. THE Platform SHALL maintain 99.9% uptime measured monthly
5. WHEN a service failure occurs, THE Platform SHALL failover to backup resources within 30 seconds
6. WHEN database queries are executed, THE Platform SHALL optimize for single-digit millisecond latency
7. WHEN users access the platform during peak hours, THE Platform SHALL maintain consistent response times
8. WHEN monitoring detects performance degradation, THE Platform SHALL alert System_Admin immediately

### Requirement 18: Multi-Channel Access

**User Story:** As a rural user with limited technology access, I want to use the platform via SMS, so that I can access healthcare services without a smartphone.

#### Acceptance Criteria

1. WHEN a user sends an SMS to the platform number, THE Platform SHALL parse the message for intent
2. WHEN an SMS command is recognized, THE Platform SHALL process it and respond via SMS within 30 seconds
3. WHEN a user requests symptom assessment via SMS, THE Platform SHALL guide them through structured questions
4. WHEN SMS responses are sent, THE Platform SHALL keep messages under 160 characters for basic phone compatibility
5. WHEN a user completes SMS-based triage, THE Platform SHALL provide Risk_Level and Care_Path via SMS
6. WHEN a user has a smartphone, THE Platform SHALL provide a Progressive Web App with offline capabilities
7. WHEN the PWA is installed, THE Platform SHALL enable push notifications for reminders and alerts
8. WHEN network connectivity is poor, THE Platform SHALL queue requests and process when connection is restored

### Requirement 19: Localization and Accessibility

**User Story:** As a rural user, I want to use the platform in my local language, so that I can understand health information clearly.

#### Acceptance Criteria

1. WHEN a user first accesses the platform, THE Platform SHALL detect or prompt for language preference
2. THE Platform SHALL support Hindi, English, Tamil, Telugu, Bengali, and Marathi languages
3. WHEN displaying medical information, THE Platform SHALL use simple, culturally appropriate terminology
4. WHEN a user changes language preference, THE Platform SHALL update all interface text immediately
5. WHEN audio guidance is available, THE Platform SHALL provide it in the user's selected language
6. WHEN forms are displayed, THE Platform SHALL support right-to-left text for applicable languages
7. WHEN images are used, THE Platform SHALL include alt text for screen readers
8. WHEN color is used to convey information, THE Platform SHALL provide alternative indicators for color-blind users

### Requirement 20: ASHA Worker Support

**User Story:** As an ASHA_Worker, I want to assist community members with platform access, so that I can help those with limited digital literacy.

#### Acceptance Criteria

1. WHEN an ASHA_Worker logs in, THE Platform SHALL provide a facilitated access mode
2. WHEN assisting a user, THE ASHA_Worker SHALL be able to initiate assessments on behalf of the user
3. WHEN an ASHA_Worker enters symptoms, THE Platform SHALL attribute the assessment to the patient, not the worker
4. WHEN an assessment is completed, THE Platform SHALL provide the ASHA_Worker with clear instructions to relay
5. WHEN an ASHA_Worker views their dashboard, THE Platform SHALL display all patients they are assisting
6. WHEN a High Risk_Level is assigned, THE Platform SHALL alert the ASHA_Worker immediately
7. WHEN an ASHA_Worker requests training materials, THE Platform SHALL provide educational resources
8. WHEN an ASHA_Worker logs activity, THE Platform SHALL track it for performance monitoring and support
