# Tutor Nation User Guide

## Overview

Tutor Nation is a comprehensive tutoring platform designed to connect students, tutors, and parents in a seamless educational ecosystem. The app facilitates personalized learning experiences through structured lessons, progress tracking, and scheduling management.

## Getting Started

### System Requirements
- iOS 18.0 or later
- Internet connection for data synchronization

### Installation
1. Download Tutor Nation from the App Store
2. Launch the app
3. Sign in or create an account

### Account Creation
Tutor Nation supports multiple authentication methods:

#### Email Registration
1. Open the app and select "Create Account"
2. Enter your first name, last name, email address, and password
3. Confirm your password
4. Tap "Create Account"

#### Social Login
- **Google Sign-In**: Tap "Sign in with Google" and follow the prompts
- **Apple Sign-In**: Tap "Sign in with Apple" and authenticate with your Apple ID

### User Roles
Upon registration, you'll be assigned a role based on your account type:
- **Student**: Access lessons, view progress, and manage your learning schedule
- **Tutor**: Manage students and track student progress
- **Parent**: Monitor your children's progress and tutoring schedules (preview interface currently available)

## Student Features

### Dashboard
After logging in, students see a tabbed interface with three main sections:

#### Lessons Tab
- View all enrolled subjects (Mathematics, Science, English, etc.)
- See progress bars for each subject showing completed vs. total lessons
- Access individual lessons with interactive content

**Navigating Lessons:**
1. Tap on a subject card to expand the lesson list
2. Select a lesson to begin
3. Answer interactive questions
4. View lesson progress

#### Calendar Tab
- View upcoming tutoring sessions
- See session details including time, duration, and subject
- Access scheduled meetings with your tutor

#### Profile Tab
- View personal information (name, grade, email)
- See overall progress across all subjects

### Lesson Experience
Lessons contain:
- Educational content tailored to your grade level
- Interactive questions with multiple choice answers
- Progress tracking
- Completion status

Example lesson topics include algebra basics, physics principles, literature analysis, and more. When you click on a lesson, you'll see example activities with interactive questions that demonstrate the lesson format.

## Tutor Features

### Dashboard
Tutors have access to three main tabs:

#### Students Tab
- View list of assigned students
- See each student's progress across subjects
- Access detailed student profiles
- Add new students to your roster

**Adding Students:**
1. Tap the "+" button in the Students tab
2. Enter the student's email address
3. The student will be linked to your tutoring sessions

#### Calendar Tab
- View your tutoring schedule
- See all scheduled sessions with students

#### Profile Tab
- View your tutor information

### Student Management
- Monitor individual student progress
- View completed and pending lessons
- Track subject-specific achievements
- View tutoring sessions

## Parent Features

**Note:** The Parent view is currently available as a user interface preview and displays mock data. Backend integration is planned for a future update, which will enable real-time access to your children's actual progress and tutoring schedules.

### Dashboard
Parents can browse through three tabs in the preview interface:

#### Schedule Tab
- Preview of how children's tutoring schedules will be displayed
- Shows lesson timelines and subject information
- Interface for tracking meeting times and subjects (backend integration pending)

#### Progress Tab
- Preview of overall progress display
- Shows subject-specific completion rates
- Displays lesson completion information (backend integration pending)

#### Children Tab
- View list of all children linked to your account
- See each child's enrolled subjects
- Access basic profile information

## Technical Features

### Data Synchronization
- All data is stored securely in the cloud
- Real-time synchronization with the backend database
- Uses AWS RDS PostgreSQL for reliable data storage

### Security
- Secure authentication with email, Google, and Apple Sign-In
- Encrypted data transmission
- Secure storage of personal information

### Architecture
- iOS native app built with SwiftUI
- Spring Boot backend for API services
- Docker containerization for reliable deployment
- AWS EC2 and RDS for cloud infrastructure

## Troubleshooting

### Common Issues

#### Login Problems
- Ensure your email and password are correct
- Check your internet connection
- Try signing in with a social provider if available

#### Data Not Loading
- Refresh the app by pulling down on the screen
- Check your internet connection
- Log out and log back in

#### Lesson Content Issues
- Ensure you're on the latest app version
- Contact support if content doesn't load properly

### Support
For technical support or questions:
- Check the FAQs section in the app
- Contact your tutor or school administrator
- Reach out to Tutor Nation support team

## Planned Features

The following features are in development and will be added in future updates:

- **AI-Generated Lessons**: Personalized lesson content generated based on student interests and learning focus
- **Parent Backend Integration**: Connect parent accounts to real student data and progress tracking
- **Admin Dashboard**: Administrative tools for school managers to view analytics and manage users
- **Advanced Scheduling**: Enhanced meeting scheduling and conflict resolution
- **Continuous Deployment**: Automated testing and deployment pipeline
- **Progress Analytics**: Detailed reports and learning insights

## Privacy and Data Policy

### Data Collection
Tutor Nation collects:
- User account information (name, email, role)
- Student progress and lesson completion data
- Scheduling and session information
- Device and usage analytics

### Data Usage
- Personal data is used to provide tutoring services
- Progress data helps personalize learning experiences
- Analytics improve app functionality and user experience

### Data Security
- All data is encrypted in transit and at rest
- Secure authentication prevents unauthorized access
- Regular security audits and updates

### Data Retention
- User data is retained as long as the account is active
- Inactive accounts may be archived after extended periods
- Users can request data deletion at any time

## Terms of Service

By using Tutor Nation, you agree to:
- Use the platform for educational purposes only
- Maintain accurate account information
- Respect the privacy of other users
- Follow community guidelines

For complete terms, please refer to the full Terms of Service document available in the app settings.

---

*This guide is for Tutor Nation version 1.0. Features and interface may vary in future updates.*