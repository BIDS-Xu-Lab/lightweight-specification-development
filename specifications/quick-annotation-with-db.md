We are building a text annotation tool for labeling clinical notes.

## Data Structure

The input data will be provided as a CSV file with four text-based columns:

1. Case ID
2. Case Title
3. Case DOI
4. Case Text

## Tech Stack

This will be a frontend-only web application, with Supabase used as the backend database and authentication provider.

### Frontend Requirements

- Use Bun, Vue, and Vite for the frontend application.
- Use Tailwind CSS for basic UI styling.
- Keep the interface clean and simple.
- Use the Supabase JavaScript SDK for:
  - user authentication and login
  - saving user annotations to the database
- Store Supabase configuration values and keys in a `.env` file.

### Backend Requirements

- The backend should be fully based on Supabase.

### Deployment

- The website will be deployed on GitHub Pages.
- A GitHub Actions workflow should be provided for deployment, along with setup instructions.

## Core Pages

The frontend should include two main pages:

1. Login Page
2. Annotation Page

### Login Page

The Login Page should use the Supabase frontend SDK to support email-and-password login.

Requirements:

- Users log in with email and password only.
- No OTP or multi-factor authentication is needed.

### Annotation Page

After login, users should be redirected to the Annotation Page. When loaded this page, display all the clinical notes to be annotated by users from supabase.

#### Layout

The page should have:

- a top navigation bar
- a three-column main layout below it

#### Navigation Bar

The navigation bar should include:

- a Save button
- a Logout button

#### Main Panel

The three columns should be:

1. Left Column
   Display the list of all notes assigned to the current user.

2. Middle Column
   Display the selected note's:
   - Title
   - DOI
   - Text

3. Right Column
   Provide three input fields for annotation, along with a short instruction such as:
   `Must not miss diagnosis`

## Data Saving

When the user clicks the Save button, the annotation data for the current note should be stored in Supabase.

## Suggested Database Design

A two-table structure is likely appropriate:

1. `notes`
   Stores all notes that need to be annotated.

2. `annotations`
   Stores each user's annotation results for each note.

### Annotation Storage Requirements

The annotation table should support:

- the user's input for each note
- a list field to store the three "Must Not Miss Diagnosis" entries
- a clear relationship between annotations, users, and notes
