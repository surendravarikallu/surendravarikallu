# Test Case Document - Skillnox Contest Platform

## 1. Document Information

| Field | Value |
|-------|-------|
| **Document Title** | Test Case Document - Skillnox Contest Platform |
| **Version** | 1.0 |
| **Date** | December 2024 |
| **Author** | QA Team |
| **Status** | Approved |

## 2. Test Strategy Overview

### 2.1 Testing Objectives
- Verify all functional requirements are met
- Ensure system performance under expected load
- Validate security measures and anti-cheat functionality
- Confirm user interface usability and accessibility
- Test system reliability and error handling

### 2.2 Testing Scope
- **In Scope**: All functional features, performance, security, usability
- **Out of Scope**: Third-party integrations, external API dependencies

### 2.3 Testing Approach
- **Unit Testing**: Individual component testing
- **Integration Testing**: Component interaction testing
- **System Testing**: End-to-end functionality testing
- **Performance Testing**: Load and stress testing
- **Security Testing**: Vulnerability and penetration testing
- **User Acceptance Testing**: Real-world scenario testing

## 3. Test Environment

### 3.1 Hardware Requirements
- **CPU**: Intel i5 or equivalent
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 50GB available space
- **Network**: Stable internet connection

### 3.2 Software Requirements
- **Operating System**: Windows 10/11, macOS 10.15+, Ubuntu 18.04+
- **Browser**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Node.js**: Version 18+
- **PostgreSQL**: Version 12+
- **Redis**: Version 6+

### 3.3 Test Data
- **Test Users**: 100+ user accounts with various roles
- **Test Contests**: 20+ contests with different configurations
- **Test Problems**: 50+ programming problems with test cases
- **Test Submissions**: 1000+ code submissions for testing

## 4. Test Cases by Module

### 4.1 Authentication Module

#### TC-AUTH-001: User Login
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-AUTH-001 |
| **Test Case Name** | User Login with Valid Credentials |
| **Priority** | High |
| **Precondition** | User account exists in system |
| **Test Steps** | 1. Navigate to login page<br>2. Enter valid username and password<br>3. Click Login button |
| **Expected Result** | User is successfully logged in and redirected to dashboard |
| **Test Data** | Username: testuser, Password: testpass123 |

#### TC-AUTH-002: User Login with Invalid Credentials
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-AUTH-002 |
| **Test Case Name** | User Login with Invalid Credentials |
| **Priority** | High |
| **Precondition** | User account exists in system |
| **Test Steps** | 1. Navigate to login page<br>2. Enter invalid username/password<br>3. Click Login button |
| **Expected Result** | Error message displayed, user remains on login page |
| **Test Data** | Username: invaliduser, Password: wrongpass |

#### TC-AUTH-003: User Logout
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-AUTH-003 |
| **Test Case Name** | User Logout |
| **Priority** | Medium |
| **Precondition** | User is logged in |
| **Test Steps** | 1. Click logout button<br>2. Confirm logout action |
| **Expected Result** | User is logged out and redirected to login page |

### 4.2 Contest Management Module

#### TC-CONTEST-001: Create Contest
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-CONTEST-001 |
| **Test Case Name** | Create New Contest |
| **Priority** | High |
| **Precondition** | Admin user is logged in |
| **Test Steps** | 1. Navigate to contest creation page<br>2. Fill in contest details<br>3. Set start and end times<br>4. Click Create Contest |
| **Expected Result** | Contest is created successfully and appears in contest list |
| **Test Data** | Title: "Test Contest", Duration: 120 minutes |

#### TC-CONTEST-002: Edit Contest
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-CONTEST-002 |
| **Test Case Name** | Edit Existing Contest |
| **Priority** | High |
| **Precondition** | Contest exists and admin is logged in |
| **Test Steps** | 1. Navigate to contest list<br>2. Click Edit on existing contest<br>3. Modify contest details<br>4. Save changes |
| **Expected Result** | Contest is updated with new details |
| **Test Data** | Updated title: "Updated Test Contest" |

#### TC-CONTEST-003: Delete Contest
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-CONTEST-003 |
| **Test Case Name** | Delete Contest |
| **Priority** | Medium |
| **Precondition** | Contest exists and admin is logged in |
| **Test Steps** | 1. Navigate to contest list<br>2. Click Delete on contest<br>3. Confirm deletion |
| **Expected Result** | Contest is deleted and removed from list |

### 4.3 Problem Management Module

#### TC-PROBLEM-001: Create Programming Problem
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-PROBLEM-001 |
| **Test Case Name** | Create Programming Problem |
| **Priority** | High |
| **Precondition** | Admin is logged in and contest exists |
| **Test Steps** | 1. Navigate to problem creation<br>2. Enter problem details<br>3. Add test cases<br>4. Set difficulty and points<br>5. Save problem |
| **Expected Result** | Problem is created and added to contest |
| **Test Data** | Title: "Sum of Two Numbers", Difficulty: Easy, Points: 100 |

#### TC-PROBLEM-002: Add Test Cases
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-PROBLEM-002 |
| **Test Case Name** | Add Test Cases to Problem |
| **Priority** | High |
| **Precondition** | Problem exists |
| **Test Steps** | 1. Open problem editor<br>2. Navigate to test cases section<br>3. Add input and expected output<br>4. Set visibility (visible/hidden)<br>5. Save test case |
| **Expected Result** | Test case is added and visible in problem |
| **Test Data** | Input: "5 3", Expected Output: "8" |

### 4.4 Code Submission Module

#### TC-SUBMIT-001: Submit Valid Code
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SUBMIT-001 |
| **Test Case Name** | Submit Valid Code Solution |
| **Priority** | High |
| **Precondition** | Student is in active contest with problem |
| **Test Steps** | 1. Select programming problem<br>2. Write correct code solution<br>3. Test against sample cases<br>4. Submit final solution |
| **Expected Result** | Code is accepted and score is calculated |
| **Test Data** | Language: JavaScript, Code: `function add(a,b){return a+b;}` |

#### TC-SUBMIT-002: Submit Invalid Code
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SUBMIT-002 |
| **Test Case Name** | Submit Invalid Code Solution |
| **Priority** | High |
| **Precondition** | Student is in active contest with problem |
| **Test Steps** | 1. Select programming problem<br>2. Write incorrect code solution<br>3. Submit solution |
| **Expected Result** | Code is rejected with appropriate error message |
| **Test Data** | Language: JavaScript, Code: `function add(a,b){return a-b;}` |

#### TC-SUBMIT-003: Code Execution Timeout
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SUBMIT-003 |
| **Test Case Name** | Code Execution Timeout |
| **Priority** | Medium |
| **Precondition** | Student submits code with infinite loop |
| **Test Steps** | 1. Write code with infinite loop<br>2. Submit solution |
| **Expected Result** | Code execution times out and appropriate error is shown |
| **Test Data** | Code: `while(true){}` |

### 4.5 Anti-Cheat Module

#### TC-ANTICHEAT-001: Tab Switch Detection
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-ANTICHEAT-001 |
| **Test Case Name** | Tab Switch Detection |
| **Priority** | High |
| **Precondition** | Student is in active contest |
| **Test Steps** | 1. Start contest participation<br>2. Switch to another browser tab<br>3. Return to contest tab |
| **Expected Result** | Tab switch is detected and warning is displayed |
| **Test Data** | Tab switches: 1 |

#### TC-ANTICHEAT-002: Multiple Tab Switches
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-ANTICHEAT-002 |
| **Test Case Name** | Multiple Tab Switches Leading to Disqualification |
| **Priority** | High |
| **Precondition** | Student is in active contest |
| **Test Steps** | 1. Start contest participation<br>2. Switch tabs 3 times<br>3. Continue contest |
| **Expected Result** | Student is disqualified after 3rd tab switch |
| **Test Data** | Tab switches: 3 (threshold) |

#### TC-ANTICHEAT-003: Fullscreen Enforcement
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-ANTICHEAT-003 |
| **Test Case Name** | Fullscreen Mode Enforcement |
| **Priority** | High |
| **Precondition** | Student attempts to join contest |
| **Test Steps** | 1. Click join contest<br>2. Deny fullscreen permission<br>3. Try to continue |
| **Expected Result** | Contest access is blocked until fullscreen is enabled |
| **Test Data** | Fullscreen permission: Denied |

### 4.6 Leaderboard Module

#### TC-LEADERBOARD-001: Real-time Updates
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-LEADERBOARD-001 |
| **Test Case Name** | Real-time Leaderboard Updates |
| **Priority** | High |
| **Precondition** | Contest is active with multiple participants |
| **Test Steps** | 1. Open leaderboard tab<br>2. Have another user submit solution<br>3. Observe leaderboard |
| **Expected Result** | Leaderboard updates automatically without refresh |
| **Test Data** | Multiple participants with submissions |

#### TC-LEADERBOARD-002: Score Calculation
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-LEADERBOARD-002 |
| **Test Case Name** | Accurate Score Calculation |
| **Priority** | High |
| **Precondition** | Multiple submissions with different scores |
| **Test Steps** | 1. Submit solutions with different scores<br>2. Check leaderboard ranking<br>3. Verify score calculations |
| **Expected Result** | Leaderboard shows correct rankings based on scores |
| **Test Data** | Scores: 100, 80, 60, 40 |

### 4.7 MCQ Module

#### TC-MCQ-001: Answer Single Choice Question
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-MCQ-001 |
| **Test Case Name** | Answer Single Choice MCQ |
| **Priority** | Medium |
| **Precondition** | Contest has MCQ questions |
| **Test Steps** | 1. Navigate to MCQ section<br>2. Read question and options<br>3. Select single answer<br>4. Submit answer |
| **Expected Result** | Answer is recorded and scored correctly |
| **Test Data** | Question: "What is 2+2?", Options: A)3 B)4 C)5 D)6, Answer: B |

#### TC-MCQ-002: Answer Multiple Choice Question
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-MCQ-002 |
| **Test Case Name** | Answer Multiple Choice MCQ |
| **Priority** | Medium |
| **Precondition** | Contest has multiple choice questions |
| **Test Steps** | 1. Navigate to MCQ section<br>2. Read question and options<br>3. Select multiple answers<br>4. Submit answers |
| **Expected Result** | Multiple answers are recorded and scored correctly |
| **Test Data** | Question: "Which are even numbers?", Options: A)2 B)3 C)4 D)5, Answer: A,C |

## 5. Performance Test Cases

### 5.1 Load Testing

#### TC-PERF-001: Concurrent User Load
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-PERF-001 |
| **Test Case Name** | Concurrent User Load Test |
| **Priority** | High |
| **Precondition** | System is deployed and ready |
| **Test Steps** | 1. Simulate 100 concurrent users<br>2. All users join same contest<br>3. All users submit solutions simultaneously<br>4. Monitor system performance |
| **Expected Result** | System handles load without degradation |
| **Test Data** | 100 concurrent users, 50 submissions/minute |

#### TC-PERF-002: Database Performance
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-PERF-002 |
| **Test Case Name** | Database Performance Test |
| **Priority** | High |
| **Precondition** | Database is populated with test data |
| **Test Steps** | 1. Execute 1000 database queries<br>2. Measure response times<br>3. Check for timeouts or errors |
| **Expected Result** | All queries complete within acceptable time |
| **Test Data** | 1000 queries, 10,000 records |

### 5.2 Stress Testing

#### TC-PERF-003: System Stress Test
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-PERF-003 |
| **Test Case Name** | System Stress Test |
| **Priority** | Medium |
| **Precondition** | System is running normally |
| **Test Steps** | 1. Gradually increase load to 200% capacity<br>2. Monitor system behavior<br>3. Identify breaking point |
| **Expected Result** | System gracefully handles overload |
| **Test Data** | 200% normal load capacity |

## 6. Security Test Cases

### 6.1 Authentication Security

#### TC-SEC-001: SQL Injection Prevention
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SEC-001 |
| **Test Case Name** | SQL Injection Prevention |
| **Priority** | High |
| **Precondition** | System is running |
| **Test Steps** | 1. Attempt SQL injection in login form<br>2. Try malicious queries in search fields<br>3. Test parameter manipulation |
| **Expected Result** | All injection attempts are blocked |
| **Test Data** | `'; DROP TABLE users; --` |

#### TC-SEC-002: XSS Prevention
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SEC-002 |
| **Test Case Name** | Cross-Site Scripting Prevention |
| **Priority** | High |
| **Precondition** | System is running |
| **Test Steps** | 1. Enter script tags in text fields<br>2. Submit malicious JavaScript code<br>3. Check for script execution |
| **Expected Result** | Scripts are sanitized and not executed |
| **Test Data** | `<script>alert('XSS')</script>` |

### 6.2 Authorization Security

#### TC-SEC-003: Unauthorized Access Prevention
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-SEC-003 |
| **Test Case Name** | Unauthorized Access Prevention |
| **Priority** | High |
| **Precondition** | System is running |
| **Test Steps** | 1. Try to access admin functions as student<br>2. Attempt to modify other users' data<br>3. Test direct URL access |
| **Expected Result** | Unauthorized access is blocked |
| **Test Data** | Student user trying to access admin panel |

## 7. Usability Test Cases

### 7.1 User Interface Testing

#### TC-UI-001: Responsive Design
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-UI-001 |
| **Test Case Name** | Responsive Design Test |
| **Priority** | Medium |
| **Precondition** | System is accessible via browser |
| **Test Steps** | 1. Test on desktop (1920x1080)<br>2. Test on tablet (768x1024)<br>3. Test on mobile (375x667) |
| **Expected Result** | Interface adapts properly to all screen sizes |
| **Test Data** | Multiple screen resolutions |

#### TC-UI-002: Accessibility Compliance
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-UI-002 |
| **Test Case Name** | Accessibility Compliance Test |
| **Priority** | Medium |
| **Precondition** | System is running |
| **Test Steps** | 1. Test with screen reader<br>2. Test keyboard navigation<br>3. Check color contrast ratios |
| **Expected Result** | System meets WCAG 2.1 guidelines |
| **Test Data** | Screen reader: NVDA, Keyboard: Tab navigation |

## 8. Integration Test Cases

### 8.1 API Integration

#### TC-INT-001: API Endpoint Testing
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-INT-001 |
| **Test Case Name** | API Endpoint Integration Test |
| **Priority** | High |
| **Precondition** | Backend API is running |
| **Test Steps** | 1. Test all GET endpoints<br>2. Test all POST endpoints<br>3. Test error handling |
| **Expected Result** | All API endpoints respond correctly |
| **Test Data** | Valid and invalid request data |

#### TC-INT-002: WebSocket Integration
| Field | Value |
|-------|-------|
| **Test Case ID** | TC-INT-002 |
| **Test Case Name** | WebSocket Real-time Communication |
| **Priority** | High |
| **Precondition** | WebSocket server is running |
| **Test Steps** | 1. Connect to WebSocket<br>2. Join contest room<br>3. Send and receive messages |
| **Expected Result** | Real-time communication works properly |
| **Test Data** | Contest ID, User ID |

## 9. Test Execution Plan

### 9.1 Test Phases

#### Phase 1: Unit Testing (Week 1-2)
- Individual component testing
- Code coverage analysis
- Bug fixing and retesting

#### Phase 2: Integration Testing (Week 3-4)
- API integration testing
- Database integration testing
- WebSocket communication testing

#### Phase 3: System Testing (Week 5-6)
- End-to-end functionality testing
- Performance testing
- Security testing

#### Phase 4: User Acceptance Testing (Week 7-8)
- Real-world scenario testing
- Usability testing
- Final validation

### 9.2 Test Environment Setup

#### Development Environment
- Local development setup
- Unit and integration tests
- Continuous integration

#### Staging Environment
- Production-like environment
- System and performance tests
- User acceptance testing

#### Production Environment
- Smoke testing
- Monitoring and validation
- Post-deployment verification

## 10. Test Data Management

### 10.1 Test Data Requirements
- **User Data**: 100+ test user accounts
- **Contest Data**: 20+ test contests with various configurations
- **Problem Data**: 50+ programming problems with test cases
- **Submission Data**: 1000+ code submissions for testing

### 10.2 Test Data Security
- Anonymized personal information
- Secure storage and access
- Regular data refresh
- Compliance with privacy regulations

## 11. Defect Management

### 11.1 Defect Classification
- **Critical**: System crash, data loss, security breach
- **High**: Major functionality not working
- **Medium**: Minor functionality issues
- **Low**: Cosmetic issues, minor improvements

### 11.2 Defect Lifecycle
1. **Discovery**: Test case execution reveals defect
2. **Reporting**: Defect logged in tracking system
3. **Assignment**: Defect assigned to development team
4. **Fix**: Developer fixes the defect
5. **Verification**: QA team verifies the fix
6. **Closure**: Defect marked as resolved

## 12. Test Metrics and Reporting

### 12.1 Test Metrics
- **Test Coverage**: Percentage of code covered by tests
- **Pass Rate**: Percentage of test cases passing
- **Defect Density**: Number of defects per module
- **Test Execution Time**: Time taken to run test suite

### 12.2 Test Reporting
- **Daily Reports**: Test execution status
- **Weekly Reports**: Progress and metrics
- **Final Report**: Overall test results and recommendations

---

**Document Version**: 1.0  
**Last Updated**: December 2024  
**Author**: QA Team  
**Review Status**: Approved
