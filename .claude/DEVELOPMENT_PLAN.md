# Memoire Audio Recording App - Development Plan

## Project Overview

Memoire is an application designed for audio recording, organization, and management. 

The prupose of the app is to enable the user to create a book of 100-250 pages based on audio recordings. Use cases are recording memoires, recording family history, recording the history of an organization.

The mobile app and web frontend allows users to 
- > record audio clips, 
- > store them securely in the cloud.
- > start the AI-powered processing
- > view output
- > publishing the final output to the marketplace 
- low priority: replaying audio clips
- low priority: optionally organize them into chapters with topics/themes, 
- low priority: respond to generated follow up questions

The web frontend allow the user to do all the things, the mobile app does, plus  
- low priority: user settings, billing, invoice history 
- low priority: editing the AI process output files

The backend does
- run the AI processing using an AI agentic system
  - load the audio recordings
  - transcribe them using a local STT model from huggingface
  - store the transcripts and mark their processing status
  - processing
      1. semantic chunking of the input text using rolling window
      2. identify the main storylines in the text by creating a graph from the textual recordings
         - each node needs to have it's own temporality (when it happend expressed as an iso format date). if the information is missing, it must be asked as a follow-up question
         - each node is connected to a list of chunks
      3. load the graph into a graph database (neo4j)
      4. use this to organize the transcripts into chapters taking into account the optional user annotation
         - identify main nodes = high number of edges
         - each main node is a chapter
         - each secondary node is attributed to a chapter by the distance (nuber fo edges) to nearest main node
         - a secondary node can be attributed to several chapters
      5. annotate the text chunks with chapter numbers
      6. If a chunk ;acks a temporal marker, ask the user.
      7. multi agent system with tools
         - chapter writer agent
            - by default loaded with main node chunks of 1 particular main node
            - tool: load adjacent nodes
            - tool: research on the web
            - tool: write chapter
         - chapter harmonizer agent
            - tool: load chapters
            - tool: edit chapter
         - follow up questions agent
            - tool: load graph
            - tool: identify nodes lacking connections
            - tool: load chunks from node
            - tool: store follow up questions based on node in markdown format
- store all the output text in markdown format 
- manage data for the UI display and marketplace


### Key Features
- User authentication (login/registration)
- Audio recording with high-quality output
- Chapter-based organization system
- Topics and themes management
- Cloud storage integration
- User settings and profile management
- Payment integration for premium features

## Technology Stack

### Mobile (Flutter)
- **Framework**: Flutter 3.x
- **State Management**: Riverpod 2.x
- **Navigation**: go_router
- **Authentication**: Firebase Auth
- **Storage**: Firebase Storage
- **Audio Recording**: record package
- **HTTP Client**: Dio
- **Secure Storage**: flutter_secure_storage

### Web Frontend
- extended mobile also in flutter but for web

### Backend
- **Platform**: Python
- **Purpose**: Data storage, manipulation, and AI components
- **Integration**: RESTful API endpoints


### Cloud Services
- **Firebase Core**: App initialization and configuration
- **Firebase Auth**: User authentication and session management
- **Firebase Storage**: Audio file storage and retrieval