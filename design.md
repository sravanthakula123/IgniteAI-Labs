# Design Document: GramaCare AI Healthcare Platform

## Overview

GramaCare AI is a serverless, AWS-native healthcare platform designed to provide AI-powered health triage and care navigation for rural communities in India. The platform addresses critical healthcare access gaps by offering multi-channel access (web, mobile, SMS) with intelligent symptom assessment, ML-based risk classification, and appropriate care routing.

### Design Principles

1. **Serverless-First Architecture**: Leverage AWS managed services to minimize operational overhead and maximize scalability
2. **Multi-Channel Access**: Support web, mobile apps, and SMS to accommodate varying levels of technology access
3. **Privacy by Design**: Implement encryption, access controls, and audit logging at every layer
4. **Offline-First for PWA**: Enable core functionality even with intermittent connectivity
5. **Performance Optimization**: Optimize for rural network conditions with compression and caching
6. **Culturally Appropriate**: Design for local languages, literacy levels, and cultural contexts
7. **Fail-Safe Defaults**: Default to higher risk levels and human review when ML confidence is low

### Technology Stack

**Frontend:**
- React.js 18+ with TypeScript for web application
- Tailwind CSS for responsive, accessible UI
- Progressive Web App (PWA) with service workers for offline support
- React Native for Android and iOS mobile applications

**Backend:**
- Node.js 18+ with TypeScript for Lambda functions (API layer)
- Python 3.11 for ML-related Lambda functions
- AWS Lambda for serverless compute
- Amazon API Gateway for REST API endpoints

**AI/ML:**
- Amazon SageMaker for hosting ML risk classification models
- scikit-learn for model training and preprocessing
- Amazon Bedrock (Claude/Titan) for LLM-powered explanations
- TensorFlow/PyTorch for deep learning models if needed

**Data Storage:**
- Amazon DynamoDB for user profiles, symptom data, assessments
- Amazon S3 for medical documents and photos
- AWS KMS for encryption key management

**Authentication & Security:**
- AWS Cognito for user identity management
- OTP-based authentication via SMS/email
- IAM roles and policies for service-to-service authentication

**Notifications & Events:**
- Amazon EventBridge for scheduling and event routing
- Amazon SNS for SMS, push notifications, and email
- Amazon SQS for asynchronous processing queues

**Monitoring & Logging:**
- Amazon CloudWatch for logs, metrics, and alarms
- AWS X-Ray for distributed tracing
- CloudWatch Dashboards for operational visibility

**DevOps:**
- AWS CodePipeline for CI/CD
- AWS CloudFormation for infrastructure as code
- Blue-green deployment strategy for zero-downtime updates

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ React PWA│  │  Android │  │   iOS    │  │   SMS    │       │
│  │   Web    │  │   App    │  │   App    │  │ Gateway  │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Amazon API Gateway (REST)                                │  │
│  │  - Authentication & Authorization                         │  │
│  │  - Request Validation & Throttling                        │  │
│  │  - CORS Configuration                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │   User   │  │  Triage  │  │   Care   │  │  Notify  │       │
│  │ Service  │  │ Service  │  │  Service │  │ Service  │       │
│  │ (Lambda) │  │ (Lambda) │  │ (Lambda) │  │ (Lambda) │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
│   AI/ML Layer    │  │  Data Layer  │  │ Event Layer  │
│ ┌──────────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │
│ │  SageMaker   │ │  │ │ DynamoDB │ │  │ │EventBridge│ │
│ │ ML Endpoint  │ │  │ │  Tables  │ │  │ │  Rules   │ │
│ └──────────────┘ │  │ └──────────┘ │  │ └──────────┘ │
│ ┌──────────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │
│ │   Bedrock    │ │  │ │    S3    │ │  │ │   SNS    │ │
│ │     LLM      │ │  │ │  Buckets │ │  │ │  Topics  │ │
│ └──────────────┘ │  │ └──────────┘ │  │ └──────────┘ │
│                  │  │ ┌──────────┐ │  │              │
│                  │  │ │   KMS    │ │  │              │
│                  │  │ │   Keys   │ │  │              │
│                  │  │ └──────────┘ │  │              │
└──────────────────┘  └──────────────┘  └──────────────┘
```

### Component Architecture

#### 1. Frontend Components

**React PWA (Web Application):**
- **Authentication Module**: Login/registration with OTP flow
- **Symptom Intake Module**: Multi-step form with conditional questions
- **Photo Upload Module**: Camera integration and file upload with compression
- **Assessment Results Module**: Display risk level, care path, and recommendations
- **Health History Module**: Timeline view of past assessments
- **Profile Management Module**: User settings, consent management, language preferences
- **Offline Support**: Service worker for caching and offline functionality

**React Native Mobile Apps:**
- Shared codebase for Android and iOS
- Native camera integration for photo capture
- Push notification support
- Biometric authentication (fingerprint/face)
- Deep linking for notifications

**SMS Interface:**
- Stateful conversation management
- Command parsing and intent recognition
- Response formatting for 160-character SMS limits
- Multi-message support for longer responses

#### 2. API Gateway Configuration

**Endpoints:**
- `POST /auth/register` - User registration
- `POST /auth/login` - OTP request
- `POST /auth/verify` - OTP verification
- `POST /auth/logout` - Session termination
- `GET /user/profile` - Get user profile
- `PUT /user/profile` - Update user profile
- `GET /user/consent` - Get consent history
- `POST /user/consent` - Update consent
- `POST /triage/assess` - Submit symptom assessment
- `GET /triage/assessment/:id` - Get assessment details
- `POST /triage/upload` - Upload medical photo
- `GET /care/recommendations/:assessmentId` - Get care path
- `POST /care/followup` - Schedule follow-up
- `GET /health/history` - Get health history
- `POST /health/summary` - Generate medical summary
- `POST /teleconsult/book` - Book teleconsultation
- `GET /teleconsult/slots` - Get available slots
- `POST /notifications/reminder` - Set medication reminder
- `GET /analytics/dashboard` - Admin analytics (admin only)

**Security Configuration:**
- AWS Cognito authorizer for all authenticated endpoints
- API key requirement for SMS gateway integration
- Rate limiting: 100 requests per minute per user
- Request size limit: 10MB for photo uploads
- CORS configuration for web and mobile origins

#### 3. Lambda Functions

**User Service (Node.js/TypeScript):**
```typescript
// Core responsibilities:
// - User registration and profile management
// - Consent record management
// - User data export (DPDP compliance)
// - User search and admin operations

interface UserProfile {
  userId: string;
  phoneNumber?: string;
  email?: string;
  preferredLanguage: string;
  dateOfBirth: string;
  gender: string;
  location: {
    state: string;
    district: string;
    pincode: string;
  };
  createdAt: string;
  updatedAt: string;
}

interface ConsentRecord {
  userId: string;
  consentId: string;
  consentVersion: string;
  consentText: string;
  agreedAt: string;
  withdrawnAt?: string;
  status: 'active' | 'withdrawn';
}

function registerUser(input: RegistrationInput): Promise<UserProfile>
function updateProfile(userId: string, updates: Partial<UserProfile>): Promise<UserProfile>
function recordConsent(userId: string, consent: ConsentInput): Promise<ConsentRecord>
function withdrawConsent(userId: string, consentId: string): Promise<ConsentRecord>
function exportUserData(userId: string): Promise<UserDataExport>
```

**Triage Service (Python):**
```python
# Core responsibilities:
# - Symptom data processing and validation
# - ML model invocation for risk classification
# - LLM integration for explanations
# - Assessment storage and retrieval

from typing import List, Dict, Optional
from enum import Enum

class RiskLevel(Enum):
    LOW = "low"
    MODERATE = "moderate"
    HIGH = "high"

class SymptomData:
    assessment_id: str
    user_id: str
    primary_symptom: str
    symptom_details: Dict[str, any]
    duration_days: int
    severity: int  # 1-10 scale
    medical_history: List[str]
    current_medications: List[str]
    photo_urls: List[str]
    timestamp: str

class RiskAssessment:
    assessment_id: str
    risk_level: RiskLevel
    confidence_score: float
    contributing_factors: List[str]
    explanation: str
    care_path: str
    requires_review: bool
    timestamp: str

def process_symptoms(symptom_data: SymptomData) -> RiskAssessment:
    """Process symptoms and invoke ML model for risk classification"""
    pass

def invoke_ml_model(features: Dict) -> tuple[RiskLevel, float]:
    """Call SageMaker endpoint for risk prediction"""
    pass

def generate_explanation(assessment: RiskAssessment, language: str) -> str:
    """Use Bedrock LLM to generate human-readable explanation"""
    pass

def store_assessment(assessment: RiskAssessment) -> None:
    """Store assessment in DynamoDB"""
    pass
```

**Care Service (Node.js/TypeScript):**
```typescript
// Core responsibilities:
// - Care path determination based on risk level
// - Recommendation generation
// - Follow-up scheduling
// - Teleconsultation booking

interface CarePath {
  assessmentId: string;
  riskLevel: 'low' | 'moderate' | 'high';
  recommendations: Recommendation[];
  nextSteps: string[];
  followUpRequired: boolean;
  followUpInterval?: number; // days
  emergencyContacts?: EmergencyContact[];
}

interface Recommendation {
  type: 'otc' | 'lifestyle' | 'monitoring' | 'emergency';
  title: string;
  description: string;
  instructions: string[];
  precautions?: string[];
}

interface FollowUp {
  followUpId: string;
  userId: string;
  assessmentId: string;
  scheduledDate: string;
  status: 'scheduled' | 'completed' | 'missed';
  reminderSent: boolean;
}

function determineCarePath(assessment: RiskAssessment): Promise<CarePath>
function scheduleFollowUp(userId: string, assessmentId: string, days: number): Promise<FollowUp>
function getAvailableSlots(providerId: string, date: string): Promise<TimeSlot[]>
function bookTeleconsultation(userId: string, slotId: string): Promise<Consultation>
```

**Notification Service (Node.js/TypeScript):**
```typescript
// Core responsibilities:
// - SMS, email, and push notification delivery
// - Medication reminder management
// - Emergency alert distribution
// - Notification preference management

interface NotificationRequest {
  userId: string;
  type: 'sms' | 'email' | 'push';
  template: string;
  variables: Record<string, string>;
  priority: 'low' | 'normal' | 'high' | 'emergency';
  scheduledAt?: string;
}

interface MedicationReminder {
  reminderId: string;
  userId: string;
  medicationName: string;
  dosage: string;
  schedule: {
    times: string[]; // ["08:00", "20:00"]
    frequency: 'daily' | 'weekly' | 'as-needed';
    startDate: string;
    endDate?: string;
  };
  adherenceLog: AdherenceEntry[];
}

interface AdherenceEntry {
  scheduledTime: string;
  takenAt?: string;
  status: 'taken' | 'missed' | 'skipped';
}

function sendNotification(request: NotificationRequest): Promise<void>
function createMedicationReminder(reminder: MedicationReminder): Promise<string>
function logAdherence(reminderId: string, entry: AdherenceEntry): Promise<void>
function sendEmergencyAlert(userId: string, assessment: RiskAssessment): Promise<void>
```

#### 4. Data Models

**DynamoDB Tables:**

**Users Table:**
```typescript
{
  PK: "USER#<userId>",
  SK: "PROFILE",
  userId: string,
  phoneNumber?: string,
  email?: string,
  preferredLanguage: string,
  dateOfBirth: string,
  gender: string,
  location: {
    state: string,
    district: string,
    pincode: string
  },
  createdAt: string,
  updatedAt: string,
  GSI1PK: "PHONE#<phoneNumber>", // For phone lookup
  GSI1SK: "USER"
}
```

**Consent Table:**
```typescript
{
  PK: "USER#<userId>",
  SK: "CONSENT#<consentId>",
  consentId: string,
  consentVersion: string,
  consentText: string,
  agreedAt: string,
  withdrawnAt?: string,
  status: 'active' | 'withdrawn'
}
```

**Assessments Table:**
```typescript
{
  PK: "USER#<userId>",
  SK: "ASSESSMENT#<timestamp>#<assessmentId>",
  assessmentId: string,
  userId: string,
  symptomData: {
    primarySymptom: string,
    details: Record<string, any>,
    duration: number,
    severity: number,
    medicalHistory: string[],
    currentMedications: string[]
  },
  riskAssessment: {
    riskLevel: 'low' | 'moderate' | 'high',
    confidenceScore: number,
    contributingFactors: string[],
    explanation: string,
    requiresReview: boolean
  },
  carePath: {
    recommendations: Recommendation[],
    nextSteps: string[],
    followUpRequired: boolean
  },
  photoUrls: string[],
  createdAt: string,
  GSI1PK: "ASSESSMENT#<assessmentId>", // For direct assessment lookup
  GSI1SK: "METADATA"
}
```

**Reminders Table:**
```typescript
{
  PK: "USER#<userId>",
  SK: "REMINDER#<reminderId>",
  reminderId: string,
  type: 'medication' | 'followup' | 'preventive',
  medicationName?: string,
  dosage?: string,
  schedule: {
    times: string[],
    frequency: string,
    startDate: string,
    endDate?: string
  },
  active: boolean,
  GSI1PK: "REMINDER#<date>", // For scheduled reminder queries
  GSI1SK: "USER#<userId>"
}
```

**Adherence Table:**
```typescript
{
  PK: "REMINDER#<reminderId>",
  SK: "ADHERENCE#<timestamp>",
  scheduledTime: string,
  takenAt?: string,
  status: 'taken' | 'missed' | 'skipped',
  notificationSent: boolean
}
```

**S3 Bucket Structure:**
```
gramacare-medical-documents/
  ├── users/
  │   └── <userId>/
  │       └── assessments/
  │           └── <assessmentId>/
  │               ├── photo1.jpg (encrypted)
  │               ├── photo2.jpg (encrypted)
  │               └── metadata.json
  └── summaries/
      └── <userId>/
          └── summary-<timestamp>.pdf (encrypted)
```

#### 5. ML Model Architecture

**Risk Classification Model:**

**Input Features:**
- Primary symptom (categorical, one-hot encoded)
- Symptom duration (numerical, days)
- Severity score (numerical, 1-10)
- Age (numerical)
- Gender (categorical)
- Medical history flags (binary features for common conditions)
- Vital signs if available (temperature, blood pressure, etc.)
- Symptom combinations (engineered features)

**Model Architecture:**
- Gradient Boosting Classifier (XGBoost or LightGBM)
- Alternative: Neural network for complex pattern recognition
- Ensemble approach combining multiple models for robustness

**Output:**
- Risk level: Low (0), Moderate (1), High (2)
- Confidence score: 0.0 to 1.0
- Feature importance scores for explainability

**Training Pipeline:**
- Historical assessment data with verified outcomes
- Regular retraining with new data (monthly)
- A/B testing for model updates
- Bias detection and mitigation for demographic fairness

**SageMaker Deployment:**
- Real-time endpoint for synchronous inference
- Auto-scaling based on request volume
- Model monitoring for drift detection
- Fallback to rule-based system if model unavailable

**LLM Integration (Amazon Bedrock):**

**Purpose:**
- Generate human-readable explanations of risk assessments
- Provide culturally appropriate health education
- Answer follow-up questions about recommendations

**Prompt Template:**
```
You are a healthcare assistant helping rural patients in India understand their health assessment.

Assessment Details:
- Primary Symptom: {symptom}
- Risk Level: {risk_level}
- Contributing Factors: {factors}

Task: Explain this assessment in simple {language} language appropriate for someone with basic health literacy. Focus on:
1. What the symptoms might indicate
2. Why this risk level was assigned
3. What the patient should do next

Keep the explanation under 200 words and avoid medical jargon.
```

**Safety Measures:**
- Content filtering for inappropriate responses
- Medical disclaimer in all LLM outputs
- Human review for high-risk cases
- Fallback to template-based responses if LLM fails

## Components and Interfaces

### Authentication Flow

**OTP-Based Login:**
```typescript
// Step 1: Request OTP
POST /auth/login
Request: {
  identifier: string, // phone or email
  type: 'phone' | 'email'
}
Response: {
  sessionId: string,
  expiresIn: number, // seconds
  message: string
}

// Step 2: Verify OTP
POST /auth/verify
Request: {
  sessionId: string,
  otp: string
}
Response: {
  accessToken: string,
  refreshToken: string,
  expiresIn: number,
  user: UserProfile
}

// Cognito Integration:
// - Custom authentication flow with Lambda triggers
// - OTP generation and validation in Lambda
// - SMS delivery via SNS
// - Token generation by Cognito
```

### Symptom Assessment Flow

**Multi-Step Assessment:**
```typescript
// Step 1: Start assessment
POST /triage/assess/start
Request: {
  primarySymptom: string
}
Response: {
  assessmentId: string,
  questions: Question[]
}

// Step 2: Submit answers
POST /triage/assess/answer
Request: {
  assessmentId: string,
  answers: Record<string, any>
}
Response: {
  nextQuestions?: Question[],
  completed: boolean
}

// Step 3: Upload photos (optional)
POST /triage/upload
Request: FormData {
  assessmentId: string,
  photo: File
}
Response: {
  photoUrl: string,
  uploadId: string
}

// Step 4: Get risk assessment
POST /triage/assess/complete
Request: {
  assessmentId: string
}
Response: {
  riskLevel: 'low' | 'moderate' | 'high',
  confidence: number,
  explanation: string,
  carePath: CarePath,
  requiresReview: boolean
}
```

### Care Path Routing Logic

```typescript
function routeCarePath(assessment: RiskAssessment): CarePath {
  const { riskLevel, confidenceScore } = assessment;
  
  // High risk: Emergency escalation
  if (riskLevel === 'high') {
    return {
      assessmentId: assessment.assessmentId,
      riskLevel: 'high',
      recommendations: [
        {
          type: 'emergency',
          title: 'Seek Immediate Medical Attention',
          description: 'Your symptoms require urgent medical evaluation',
          instructions: [
            'Call emergency services (108) immediately',
            'Go to the nearest hospital emergency department',
            'Do not delay seeking care'
          ]
        }
      ],
      nextSteps: [
        'Contact emergency services',
        'Inform family members',
        'Bring any current medications with you'
      ],
      followUpRequired: true,
      followUpInterval: 1,
      emergencyContacts: getEmergencyContacts(assessment.userId)
    };
  }
  
  // Moderate risk: Monitor and advise
  if (riskLevel === 'moderate') {
    return {
      assessmentId: assessment.assessmentId,
      riskLevel: 'moderate',
      recommendations: [
        {
          type: 'monitoring',
          title: 'Monitor Your Symptoms',
          description: 'Your symptoms need attention but are not immediately urgent',
          instructions: [
            'Monitor symptoms for the next 24-48 hours',
            'Seek medical care if symptoms worsen',
            'Follow the care instructions provided'
          ]
        },
        ...generateModerateRecommendations(assessment)
      ],
      nextSteps: [
        'Follow monitoring instructions',
        'Complete follow-up assessment in 2 days',
        'Contact healthcare provider if symptoms worsen'
      ],
      followUpRequired: true,
      followUpInterval: 2
    };
  }
  
  // Low risk: Self-care guidance
  return {
    assessmentId: assessment.assessmentId,
    riskLevel: 'low',
    recommendations: [
      {
        type: 'otc',
        title: 'Self-Care Recommendations',
        description: 'Your symptoms can likely be managed at home',
        instructions: generateLowRiskInstructions(assessment),
        precautions: [
          'Seek medical care if symptoms persist beyond 7 days',
          'Watch for warning signs of worsening condition'
        ]
      }
    ],
    nextSteps: [
      'Follow self-care instructions',
      'Rest and stay hydrated',
      'Monitor for improvement'
    ],
    followUpRequired: true,
    followUpInterval: 7
  };
}
```

### Event-Driven Architecture

**EventBridge Rules:**

```typescript
// Follow-up reminder rule
{
  "source": ["gramacare.followup"],
  "detail-type": ["FollowUp Scheduled"],
  "detail": {
    "followUpDate": [{ "exists": true }]
  }
}
// Target: Lambda function to send reminder

// Medication reminder rule (scheduled)
{
  "schedule": "rate(5 minutes)" // Check for due reminders
}
// Target: Lambda function to process due reminders

// Emergency alert rule
{
  "source": ["gramacare.triage"],
  "detail-type": ["High Risk Assessment"],
  "detail": {
    "riskLevel": ["high"]
  }
}
// Target: SNS topic for emergency notifications
```

**Event Payloads:**

```typescript
// Follow-up scheduled event
{
  "version": "0",
  "id": "event-id",
  "detail-type": "FollowUp Scheduled",
  "source": "gramacare.followup",
  "time": "2024-01-15T10:00:00Z",
  "detail": {
    "userId": "user-123",
    "assessmentId": "assess-456",
    "followUpId": "followup-789",
    "followUpDate": "2024-01-17T10:00:00Z"
  }
}

// High risk assessment event
{
  "version": "0",
  "id": "event-id",
  "detail-type": "High Risk Assessment",
  "source": "gramacare.triage",
  "time": "2024-01-15T10:00:00Z",
  "detail": {
    "userId": "user-123",
    "assessmentId": "assess-456",
    "riskLevel": "high",
    "primarySymptom": "chest pain",
    "confidence": 0.92
  }
}
```

### SMS Interface Design

**Command Structure:**
```
// Registration
SMS: "JOIN GRAMACARE"
Response: "Welcome! Reply with your age and gender (e.g., 25 M)"

// Symptom reporting
SMS: "SYMPTOM fever headache"
Response: "How many days? Reply with number"
SMS: "3"
Response: "Rate severity 1-10"
SMS: "7"
Response: "Assessing... Your risk: MODERATE. Monitor symptoms. Call 108 if worse. Details: [link]"

// Follow-up
SMS: "FOLLOWUP assess-456"
Response: "How are you feeling? Reply: BETTER, SAME, or WORSE"
SMS: "BETTER"
Response: "Great! Continue rest and hydration. Next check: 2 days"

// Help
SMS: "HELP"
Response: "Commands: SYMPTOM, FOLLOWUP, HISTORY, HELP. For emergency call 108"
```

**SMS State Management:**
```typescript
interface SMSSession {
  phoneNumber: string;
  currentFlow: 'registration' | 'symptom' | 'followup' | null;
  state: Record<string, any>;
  lastMessageAt: string;
  expiresAt: string; // 15 minutes timeout
}

// Store in DynamoDB with TTL for automatic cleanup
```

