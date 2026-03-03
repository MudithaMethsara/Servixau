# ServixAU Project Comprehensive Implementation Plan

This document provides a granular, code-level roadmap for the completion of the ServixAU platform. It is based on a deep analysis of the existing Spring Boot backend and Expo/React Native frontend.

---

## 1. Core Authentication & Identity (Status: 70% Complete)

### Backend (`servixau-backend`)
- [ ] **JWT Refresh Token Mechanism**:
    - Update `JwtTokenProvider` to generate and validate refresh tokens.
    - Create a `/api/auth/refresh` endpoint in `AuthController`.
    - Logic: Check if the refresh token is valid and not expired; issue a new access token.
- [ ] **Production-Ready SMS Integration**:
    - Replace current placeholder logic in `SmsServiceImpl.java` with a real Twilio client.
    - Configuration: Ensure `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, and `TWILIO_FROM_NUMBER` are in `.env`.
- [ ] **Role-Based Access Control (RBAC)**:
    - Verify all `@PreAuthorize` annotations in controllers (e.g., `JobRequestController` should only allow `ROLE_CLIENT`).
    - Ensure `SecurityConfig.java` properly filters requests based on `UserRole` enum.
- [ ] **Security Auditing**:
    - Implement an `AuditLogService`.
    - Logic: Save records to the `audit_logs` table for every login, failed login, and registration event.

### Frontend (`servixau-app`)
- [ ] **Dynamic Navigation Guard**:
    - Refine `AuthContext.tsx` to handle role-based redirection.
    - Logic: If `user.role === 'ROLE_CLIENT'`, redirect to `/client/dashboard`; if `ROLE_SERVICE_PROVIDER`, redirect to `/provider/dashboard`.
- [ ] **Token Interceptor**:
    - Update `services/api.ts` to include an `Authorization` header in all requests.
    - Logic: Automatically attempt to refresh the token if a `401 Unauthorized` is received.
- [ ] **Input Validation Layer**:
    - Integrate `react-hook-form` with `zod` for all registration screens.
    - Files: `signup-steps/step1.tsx`, `service-provider-signup/step1.tsx`, etc.

---

## 2. Service Provider Profile & Verification (Status: 50% Complete)

### Backend
- [ ] **Provider Profile Completion**:
    - Update `ProviderService` to support saving bio, experience years, and service radius.
    - Create `ProviderAvailability` entity and corresponding repository/service.
- [ ] **Admin Verification API**:
    - Create `AdminController` with an endpoint: `POST /api/admin/verify-document/{docId}`.
    - Logic: Update `VerificationStatus` of the document and potentially the `UserStatus` to `ACTIVE` once mandatory docs are approved.

### Frontend
- [ ] **Provider Profile Editor**:
    - Build a UI for providers to manage their skills (already started in `select-skills.tsx`).
    - Implement document re-upload logic if an admin rejects a document.
- [ ] **ABN Real-time Validation**:
    - Connect the `verify-abn` button in the UI to the backend's `AbnVerificationServiceImpl`.
    - Logic: Update the `SignupContext` with the returned company name and status.

---

## 3. Job Request System (Status: 30% Complete)

### Backend
- [ ] **Job Lifecycle State Machine**:
    - Implement `JobService` logic to move a job from `PENDING` to `ASSIGNED`.
    - Logic: When a provider accepts a job, update `job_assignments` table and job status.
- [ ] **Matching & Notification**:
    - Create a logic that queries `ServiceProvider` entities matching the `ServiceCategory` of a new `Job`.
    - Logic: Trigger `NotificationService` to send push notifications to those providers.
- [ ] **Checklist & Evidence**:
    - Implement `JobChecklistService`.
    - Logic: Providers must check off items and potentially upload a photo (to S3) before they can "Complete" a job.

### Frontend
- [ ] **Client: Job Posting Wizard**:
    - Create a multi-step form: Category -> Details -> Schedule -> Budget -> Review.
    - Use `JobRequestCreateRequest` DTO structure.
- [ ] **Provider: Job Feed**:
    - Build a "Marketplace" screen showing jobs in the provider's area/category.
    - Implement "Accept Job" and "Decline Job" buttons.
- [ ] **Job Tracking View**:
    - Implement a "Clock-in" button that captures current GPS coordinates and sends them to the `/api/attendance` endpoint.

---

## 4. File Management & AWS S3 (Status: 40% Complete)

### Backend
- [ ] **Pre-signed URL Generator**:
    - Update `S3Service.java` to generate temporary URLs (e.g., 15-minute expiry) for private documents.
    - Controller: `GET /api/files/download/{fileId}`.
- [ ] **S3 File Cleanup**:
    - Implement a `@Scheduled` task in Spring Boot to delete objects from S3 that exist in the bucket but have no record in the `documents` database table.

### Frontend
- [ ] **S3 Upload Component**:
    - Create a reusable hook `useFileUpload` that handles the 2-step process: 1) Upload to S3, 2) Send metadata to backend.
    - Logic: Use `expo-document-picker` or `expo-image-picker`.

---

## 5. Financials, Invoices & Payments (Status: 10% Complete)

### Backend
- [ ] **Invoice Generation Engine**:
    - Create a service to generate PDF invoices (using iText7 already in `pom.xml`).
    - Logic: Calculate `hourlyRate * hoursWorked`, add GST, and generate a record in the `invoices` table.
- [ ] **Payment Gateway Integration**:
    - Implement `StripeService`.
    - Logic: Handle `PaymentIntent` creation and webhook listening for payment confirmation.

### Frontend
- [ ] **Invoice List & PDF Viewer**:
    - Allow users to see a list of past jobs and download the corresponding invoice.
- [ ] **Payment UI**:
    - Integrate Stripe's mobile SDK for secure credit card entry.

---

## 6. Communication & Real-time (Status: 0% Complete)

### Backend
- [ ] **Messaging API**:
    - Implement `MessageService` for the `messages` table.
    - Logic: Support threading between `clientId` and `providerId` for a specific `jobId`.
- [ ] **Push Notification Service**:
    - Integrate with Expo's Push Notification API.
    - Logic: Send alerts for new jobs, new messages, and job status changes.

### Frontend
- [ ] **Chat UI**:
    - Create a standard chat interface with message bubbles and timestamps.
- [ ] **Notification Center**:
    - A dedicated tab to see all system alerts and updates.

---

## 7. Database & Infrastructure (Status: 60% Complete)

- [ ] **Full Liquibase Migration**:
    - Review `db.changelog-master.xml` to ensure all 28 entities (Invoices, Attendance, GPS logs, etc.) have their tables defined.
- [ ] **Docker Deployment**:
    - Finalize the `docker-compose.yml` to include the Spring Boot app, MySQL, and a Redis instance for caching JWT blacklists (if implemented).
- [ ] **Environment Validation**:
    - Create a script to verify all required `.env` keys (AWS, Twilio, ABR, Stripe) are present before the app starts.

---

## 8. Final Testing & Validation

- [ ] **Backend Unit Tests**: Reach 80% coverage on `AuthService`, `JobService`, and `ProviderService`.
- [ ] **Frontend E2E Tests**: (Optional) Use Detox or similar to test the registration and job booking flow.
- [ ] **Swagger Documentation**: Complete all `@Schema` and `@Operation` tags in controllers to ensure the API is fully discoverable.
