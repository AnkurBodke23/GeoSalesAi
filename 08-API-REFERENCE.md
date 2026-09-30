# NearBiz AI-Powered B2B Sales Platform - API Reference

This document provides a comprehensive API reference for the NearBiz platform, organized by domain. 

## 1. Authentication (`/api/auth/*`)

### POST /api/auth/register
**Auth:** None | **Roles:** None
**Description:** Create a new user account.
**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "full_name": "John Doe",
  "role": "user",
  "phone": "+1234567890"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Registration successful. Please verify your email.",
  "user": {
    "id": "u_123",
    "email": "user@example.com",
    "full_name": "John Doe",
    "role": "user"
  }
}
```
**Notes:** Sends verification email asynchronously.

### POST /api/auth/login
**Auth:** None | **Roles:** None
**Description:** Authenticate user and return tokens.
**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```
**Response:**
```json
{
  "success": true,
  "access_token": "eyJhbGciOi...",
  "user": {
    "id": "u_123",
    "email": "user@example.com",
    "role": "user"
  }
}
```
**Notes:** Sets `refresh_token` as an HTTP-only secure cookie.

### POST /api/auth/refresh
**Auth:** None | **Roles:** None
**Description:** Exchange refresh cookie for a new access token.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "access_token": "eyJhbGciOi..."
}
```
**Notes:** Requires valid `refresh_token` cookie.

### POST /api/auth/logout
**Auth:** Required | **Roles:** Any
**Description:** Revoke refresh token and clear cookies.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```
**Notes:** Clears `refresh_token` cookie.

### GET /api/auth/me
**Auth:** Required | **Roles:** Any
**Description:** Get the current authenticated user's profile summary.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "user": {
    "id": "u_123",
    "email": "user@example.com",
    "full_name": "John Doe",
    "role": "user",
    "company_id": "c_456"
  }
}
```
**Notes:** Lightweight profile for UI state.

### POST /api/auth/forgot-password
**Auth:** None | **Roles:** None
**Description:** Request a password reset link.
**Request Body:**
```json
{
  "email": "user@example.com"
}
```
**Response:**
```json
{
  "success": true,
  "message": "If an account exists, a reset link has been sent."
}
```
**Notes:** Always returns success to prevent email enumeration.

### POST /api/auth/reset-password
**Auth:** None | **Roles:** None
**Description:** Apply a new password using a reset token.
**Request Body:**
```json
{
  "token": "reset_token_abc123",
  "new_password": "NewSecurePassword123!"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Password reset successfully"
}
```
**Notes:** Tokens expire after 1 hour.

### POST /api/auth/verify-email
**Auth:** None | **Roles:** None
**Description:** Verify a user's email address using a token.
**Request Body:**
```json
{
  "token": "verify_token_xyz789"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Email verified successfully"
}
```
**Notes:** Activates the user account.

## 2. Users (`/api/users/*`)

### GET /api/users/me
**Auth:** Required | **Roles:** Any
**Description:** Retrieve the full profile of the authenticated user, including company info.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": {
    "id": "u_123",
    "email": "user@example.com",
    "full_name": "John Doe",
    "phone": "+1234567890",
    "avatar": "https://url.to/avatar.jpg",
    "preferences": {
      "email_notifications": true
    },
    "company": {
      "id": "c_456",
      "name": "Tech Corp"
    }
  }
}
```
**Notes:** N/A

### PUT /api/users/me
**Auth:** Required | **Roles:** Any
**Description:** Update user profile information.
**Request Body:**
```json
{
  "full_name": "John Doe Updated",
  "phone": "+1987654321",
  "preferences": {
    "email_notifications": false
  }
}
```
**Response:**
```json
{
  "success": true,
  "data": {
    "id": "u_123",
    "full_name": "John Doe Updated"
  }
}
```
**Notes:** Cannot update email or role through this endpoint.

### GET /api/users/me/notifications
**Auth:** Required | **Roles:** Any
**Description:** Fetch paginated notifications for the user.
**Request Body/Query:**
```json
{
  "page": 1,
  "limit": 20
}
```
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "notif_1",
      "type": "meeting_request",
      "message": "New meeting request from Jane Smith",
      "is_read": false,
      "created_at": "2024-01-10T12:00:00Z"
    }
  ],
  "pagination": {
    "total": 5,
    "page": 1,
    "pages": 1
  }
}
```
**Notes:** Sorted by created_at DESC.

### PUT /api/users/me/notifications/read-all
**Auth:** Required | **Roles:** Any
**Description:** Mark all user notifications as read.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "All notifications marked as read"
}
```
**Notes:** N/A

### GET /api/users/me/followed-companies
**Auth:** Required | **Roles:** Any
**Description:** List of companies the user follows.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "c_456",
      "name": "Tech Corp",
      "logo": "https://url.to/logo.png"
    }
  ]
}
```
**Notes:** N/A

### GET /api/users/me/meetings
**Auth:** Required | **Roles:** salesperson, company_admin
**Description:** Get upcoming meetings assigned to the user as a salesperson.
**Request Body/Query:**
```json
{
  "status": "scheduled"
}
```
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "meet_1",
      "title": "Product Demo",
      "start_time": "2024-01-15T10:00:00Z",
      "with_user": { "name": "Jane Smith" }
    }
  ]
}
```
**Notes:** Filters available by status (scheduled, completed, cancelled).

### GET /api/users/me/leads
**Auth:** Required | **Roles:** salesperson, company_admin
**Description:** Get leads assigned to the user.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "lead_1",
      "company_name": "Client Inc",
      "stage": "contacted",
      "score": 85
    }
  ]
}
```
**Notes:** N/A

## 3. Companies (`/api/companies/*`)

### GET /api/companies
**Auth:** Optional | **Roles:** Any
**Description:** Search and list companies.
**Request Body/Query:**
```json
{
  "q": "tech",
  "industry": "Software",
  "city": "San Francisco",
  "verified": "true",
  "page": 1,
  "limit": 10
}
```
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "c_1",
      "name": "Tech Corp",
      "industry": "Software",
      "verified": true
    }
  ],
  "pagination": { "total": 100, "page": 1, "pages": 10 }
}
```
**Notes:** N/A

### POST /api/companies
**Auth:** Required | **Roles:** platform_admin, user (self-registration)
**Description:** Create a new company profile.
**Request Body:**
```json
{
  "name": "NewBiz LLC",
  "industry": "Manufacturing",
  "description": "We make things.",
  "location": {
    "address": "123 Main St",
    "city": "NY"
  }
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "c_new", "name": "NewBiz LLC" }
}
```
**Notes:** Creator is automatically assigned `company_admin` role.

### GET /api/companies/:id
**Auth:** Optional | **Roles:** Any
**Description:** Fetch full company profile by ID.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": {
    "id": "c_1",
    "name": "Tech Corp",
    "description": "...",
    "follower_count": 150
  }
}
```
**Notes:** Includes public stats.

### GET /api/companies/slug/:slug
**Auth:** Optional | **Roles:** Any
**Description:** Fetch company profile by URL slug.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "c_1", "slug": "tech-corp", "name": "Tech Corp" }
}
```
**Notes:** Used for SEO-friendly public profiles.

### PUT /api/companies/:id
**Auth:** Required | **Roles:** company_admin, platform_admin
**Description:** Update company profile details.
**Request Body:**
```json
{
  "description": "Updated description",
  "website": "https://newbiz.example.com"
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "c_1", "website": "https://newbiz.example.com" }
}
```
**Notes:** N/A

### GET /api/companies/:id/products
**Auth:** Optional | **Roles:** Any
**Description:** Retrieve the product catalogue for a company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "p_1", "name": "Widget A", "price": 99.99 }
  ]
}
```
**Notes:** N/A

### POST /api/companies/:id/products
**Auth:** Required | **Roles:** company_admin, company_member
**Description:** Add a new product to the catalogue.
**Request Body:**
```json
{
  "name": "Widget B",
  "description": "A new widget",
  "price": 149.99,
  "category": "Hardware"
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "p_2", "name": "Widget B" }
}
```
**Notes:** N/A

### PUT /api/companies/:id/products/:pid
**Auth:** Required | **Roles:** company_admin, company_member
**Description:** Update an existing product.
**Request Body:**
```json
{
  "price": 129.99
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "p_2", "price": 129.99 }
}
```
**Notes:** N/A

### DELETE /api/companies/:id/products/:pid
**Auth:** Required | **Roles:** company_admin, company_member
**Description:** Delete a product from the catalogue.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```
**Notes:** Soft delete recommended in backend.

### GET /api/companies/:id/posts
**Auth:** Optional | **Roles:** Any
**Description:** Get post history for a company.
**Request Body/Query:**
```json
{ "page": 1, "limit": 10 }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "post_1", "content": "Hello world", "likes": 5 }
  ]
}
```
**Notes:** N/A

### GET /api/companies/:id/followers
**Auth:** Optional | **Roles:** Any
**Description:** Get list of users following the company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "count": 150,
  "data": [
    { "id": "u_1", "name": "Alice" }
  ]
}
```
**Notes:** N/A

### GET /api/companies/:id/members
**Auth:** Required | **Roles:** company_admin, platform_admin
**Description:** Get team members belonging to the company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "u_2", "name": "Bob", "role": "salesperson" }
  ]
}
```
**Notes:** Internal view only.

### POST /api/companies/:id/members
**Auth:** Required | **Roles:** company_admin
**Description:** Invite/add a user to the company team.
**Request Body:**
```json
{
  "email": "charlie@techcorp.com",
  "role": "company_member"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Invitation sent"
}
```
**Notes:** Sends an email invite.

### DELETE /api/companies/:id/members/:uid
**Auth:** Required | **Roles:** company_admin
**Description:** Remove a member from the company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Member removed"
}
```
**Notes:** Revokes company access for the user.

### POST /api/companies/:id/follow
**Auth:** Required | **Roles:** Any
**Description:** Follow a company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Following company"
}
```
**Notes:** N/A

### DELETE /api/companies/:id/follow
**Auth:** Required | **Roles:** Any
**Description:** Unfollow a company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Unfollowed company"
}
```
**Notes:** N/A

### GET /api/companies/:id/is-following
**Auth:** Required | **Roles:** Any
**Description:** Check if current user is following the company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "is_following": true
}
```
**Notes:** Fast lookup for UI state.

### GET /api/companies/:id/analytics
**Auth:** Required | **Roles:** company_admin
**Description:** Retrieve analytics dashboard data.
**Request Body/Query:**
```json
{ "period": "30d" }
```
**Response:**
```json
{
  "success": true,
  "data": {
    "profile_views": 1200,
    "post_engagement": 350,
    "leads_generated": 15
  }
}
```
**Notes:** N/A

### GET /api/companies/:id/meetings
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Get meeting history for the company.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "m_1", "title": "Discovery Call", "date": "2024-01-05" }
  ]
}
```
**Notes:** N/A


## 4. Products (`/api/products/*`)

### GET /api/products
**Auth:** Optional | **Roles:** Any
**Description:** Search products across all companies globally.
**Request Body/Query:**
```json
{ "q": "laptop", "category": "Electronics", "min_price": 500 }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "p_10", "name": "Pro Laptop", "company": { "id": "c_2", "name": "Tech Corp" } }
  ]
}
```
**Notes:** N/A

### GET /api/products/:id
**Auth:** Optional | **Roles:** Any
**Description:** Get detailed information for a specific product.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "p_10", "name": "Pro Laptop", "price": 1200 }
}
```
**Notes:** N/A

### POST /api/products/:id/inquiry
**Auth:** Required | **Roles:** Any
**Description:** Send a product inquiry (generates a lead/message).
**Request Body:**
```json
{ "message": "I would like to order 50 units." }
```
**Response:**
```json
{
  "success": true,
  "message": "Inquiry sent successfully"
}
```
**Notes:** Automatically creates a conversation or lead in CRM.


## 5. Discovery & Map (`/api/discovery/*`)

### GET /api/discovery/nearby
**Auth:** Optional | **Roles:** Any
**Description:** Find companies within a radius.
**Request Body/Query:**
```json
{
  "lat": 37.7749,
  "lng": -122.4194,
  "radius_km": 10,
  "industry": "Tech"
}
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "c_1", "name": "Tech Corp", "distance_km": 2.5, "lat": 37.775, "lng": -122.410 }
  ]
}
```
**Notes:** Uses PostGIS or MongoDB Geospatial queries.

### GET /api/discovery/bounds
**Auth:** Optional | **Roles:** Any
**Description:** Find companies within a map viewport (bounding box).
**Request Body/Query:**
```json
{
  "sw_lat": 37.7, "sw_lng": -122.5,
  "ne_lat": 37.8, "ne_lng": -122.3
}
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "c_1", "lat": 37.775, "lng": -122.410 }
  ]
}
```
**Notes:** Optimized for map panning.

### GET /api/discovery/search
**Auth:** Optional | **Roles:** Any
**Description:** Full-text business search optimized for discovery.
**Request Body/Query:**
```json
{ "q": "software solutions" }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "c_1", "name": "Tech Corp" }
  ]
}
```
**Notes:** May utilize Elasticsearch or similar.

### POST /api/discovery/route
**Auth:** Required | **Roles:** Any
**Description:** Point-to-point routing via OSRM.
**Request Body:**
```json
{
  "start": { "lat": 37.7, "lng": -122.4 },
  "end": { "lat": 37.8, "lng": -122.3 }
}
```
**Response:**
```json
{
  "success": true,
  "route": {
    "distance_meters": 12500,
    "duration_seconds": 900,
    "geometry": "encoded_polyline_string"
  }
}
```
**Notes:** N/A

### POST /api/discovery/trip
**Auth:** Required | **Roles:** Any
**Description:** Multi-stop route optimization (TSP) via OSRM.
**Request Body:**
```json
{
  "stops": [
    { "lat": 37.7, "lng": -122.4 },
    { "lat": 37.75, "lng": -122.45 },
    { "lat": 37.8, "lng": -122.3 }
  ]
}
```
**Response:**
```json
{
  "success": true,
  "trip": {
    "ordered_stops": [0, 2, 1],
    "geometry": "encoded_polyline_string"
  }
}
```
**Notes:** Solves Traveling Salesperson Problem.

### GET /api/discovery/geocode
**Auth:** Optional | **Roles:** Any
**Description:** Address to coordinates using Photon proxy.
**Request Body/Query:**
```json
{ "address": "123 Main St, San Francisco" }
```
**Response:**
```json
{
  "success": true,
  "data": { "lat": 37.7749, "lng": -122.4194 }
}
```
**Notes:** N/A


## 6. Feed & Posts (`/api/feed/*`, `/api/posts/*`)

### GET /api/feed
**Auth:** Required | **Roles:** Any
**Description:** Personalized feed from followed companies.
**Request Body/Query:**
```json
{ "cursor": "timestamp_or_id", "limit": 20 }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "post_1", "company": { "name": "Tech Corp" }, "content": "New product launch!" }
  ],
  "next_cursor": "post_20_timestamp"
}
```
**Notes:** Cursor-paginated for infinite scroll.

### GET /api/feed/discover
**Auth:** Optional | **Roles:** Any
**Description:** Public discovery feed showing trending posts.
**Request Body/Query:**
```json
{ "page": 1 }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "post_trending", "likes": 500 } ]
}
```
**Notes:** N/A

### POST /api/posts
**Auth:** Required | **Roles:** company_member, company_admin
**Description:** Create a new company post.
**Request Body:**
```json
{
  "content": "Check out our new AI feature!",
  "media_urls": ["https://url.to/image.jpg"]
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "post_new", "content": "Check out our new AI feature!" }
}
```
**Notes:** Automatically tied to the creator's company.

### GET /api/posts/:id
**Auth:** Optional | **Roles:** Any
**Description:** Get post detail (increments view_count).
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "post_1", "views": 105 }
}
```
**Notes:** View counting should be debounced.

### PUT /api/posts/:id
**Auth:** Required | **Roles:** company_member, company_admin
**Description:** Update a post.
**Request Body:**
```json
{ "content": "Updated content" }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "post_1", "content": "Updated content" }
}
```
**Notes:** N/A

### DELETE /api/posts/:id
**Auth:** Required | **Roles:** company_member, company_admin
**Description:** Delete a post.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Post deleted"
}
```
**Notes:** N/A

### POST /api/posts/:id/like
**Auth:** Required | **Roles:** Any
**Description:** Like a post.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Post liked"
}
```
**Notes:** N/A

### DELETE /api/posts/:id/like
**Auth:** Required | **Roles:** Any
**Description:** Unlike a post.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Post unliked"
}
```
**Notes:** N/A

### GET /api/posts/:id/comments
**Auth:** Optional | **Roles:** Any
**Description:** Get threaded comments for a post.
**Request Body/Query:**
```json
{ "page": 1 }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "c_1", "user": "Alice", "text": "Great post!" }
  ]
}
```
**Notes:** N/A

### POST /api/posts/:id/comments
**Auth:** Required | **Roles:** Any
**Description:** Add a comment to a post.
**Request Body:**
```json
{ "text": "I agree!" }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "c_2", "text": "I agree!" }
}
```
**Notes:** N/A

### DELETE /api/posts/:id/comments/:cid
**Auth:** Required | **Roles:** Comment author, company_admin
**Description:** Soft-delete a comment.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Comment deleted"
}
```
**Notes:** Marks comment text as "[Deleted]".

### POST /api/posts/:id/media
**Auth:** Required | **Roles:** company_member, company_admin
**Description:** Attach media to a post.
**Request Body:**
```json
{ "media_url": "https://url.to/video.mp4", "type": "video" }
```
**Response:**
```json
{
  "success": true,
  "message": "Media attached"
}
```
**Notes:** Expects pre-uploaded URL from `/api/upload`.


## 7. Messaging (`/api/conversations/*`)

### GET /api/conversations
**Auth:** Required | **Roles:** Any
**Description:** Get user's conversations.
**Request Body/Query:**
```json
{ "page": 1, "limit": 20 }
```
**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "conv_1", "participants": ["u_1", "u_2"], "last_message": "Hello" }
  ]
}
```
**Notes:** N/A

### POST /api/conversations
**Auth:** Required | **Roles:** Any
**Description:** Start a new conversation.
**Request Body:**
```json
{ "recipient_id": "u_2", "initial_message": "Hi there!" }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "conv_2" }
}
```
**Notes:** N/A

### GET /api/conversations/:id
**Auth:** Required | **Roles:** Participant
**Description:** Get conversation details and messages.
**Request Body/Query:**
```json
{ "page": 1 }
```
**Response:**
```json
{
  "success": true,
  "data": {
    "id": "conv_1",
    "messages": [
      { "id": "msg_1", "sender_id": "u_1", "content": "Hello" }
    ]
  }
}
```
**Notes:** N/A

### POST /api/conversations/:id/messages
**Auth:** Required | **Roles:** Participant
**Description:** Send a message in a conversation.
**Request Body:**
```json
{ "content": "How are you?" }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "msg_2", "content": "How are you?" }
}
```
**Notes:** Triggers WebSocket event.

### PUT /api/conversations/:id/read
**Auth:** Required | **Roles:** Participant
**Description:** Mark conversation as read up to a certain message.
**Request Body:**
```json
{ "last_read_message_id": "msg_2" }
```
**Response:**
```json
{
  "success": true
}
```
**Notes:** N/A


## 8. Meetings (`/api/meeting-requests/*`, `/api/meetings/*`)

### POST /api/meeting-requests
**Auth:** Required | **Roles:** Any
**Description:** Create a meeting request.
**Request Body:**
```json
{
  "recipient_id": "u_2",
  "proposed_times": ["2024-02-01T10:00:00Z", "2024-02-01T14:00:00Z"],
  "agenda": "Discuss product integration"
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "req_1", "status": "pending" }
}
```
**Notes:** N/A

### GET /api/meeting-requests
**Auth:** Required | **Roles:** Any
**Description:** Get user's meeting requests (sent + received).
**Request Body/Query:**
```json
{ "type": "received" }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "req_1", "agenda": "Discuss..." } ]
}
```
**Notes:** N/A

### GET /api/meeting-requests/:id
**Auth:** Required | **Roles:** Participant
**Description:** Get request detail.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "req_1", "status": "pending" }
}
```
**Notes:** N/A

### PATCH /api/meeting-requests/:id/accept
**Auth:** Required | **Roles:** Recipient
**Description:** Accept a meeting request.
**Request Body:**
```json
{ "selected_time": "2024-02-01T10:00:00Z" }
```
**Response:**
```json
{
  "success": true,
  "message": "Meeting scheduled",
  "meeting_id": "m_1"
}
```
**Notes:** Converts request into a scheduled Meeting.

### PATCH /api/meeting-requests/:id/decline
**Auth:** Required | **Roles:** Recipient
**Description:** Decline a request.
**Request Body:**
```json
{ "reason": "Not interested at the moment" }
```
**Response:**
```json
{
  "success": true,
  "message": "Request declined"
}
```
**Notes:** N/A

### PATCH /api/meeting-requests/:id/counter
**Auth:** Required | **Roles:** Recipient
**Description:** Propose alternative times.
**Request Body:**
```json
{ "proposed_times": ["2024-02-02T11:00:00Z"] }
```
**Response:**
```json
{
  "success": true,
  "message": "Counter-proposal sent"
}
```
**Notes:** N/A

### GET /api/meetings
**Auth:** Required | **Roles:** Any
**Description:** Get confirmed meetings.
**Request Body/Query:**
```json
{ "status": "upcoming" }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "m_1", "start_time": "2024-02-01T10:00:00Z" } ]
}
```
**Notes:** N/A

### GET /api/meetings/:id
**Auth:** Required | **Roles:** Participant
**Description:** Get meeting detail.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "m_1", "link": "https://zoom.us/j/123..." }
}
```
**Notes:** N/A

### PATCH /api/meetings/:id/complete
**Auth:** Required | **Roles:** Participant
**Description:** Mark meeting as completed.
**Request Body:**
```json
{ "outcome": "positive", "notes": "Client interested in Pro tier." }
```
**Response:**
```json
{
  "success": true,
  "message": "Meeting completed"
}
```
**Notes:** Updates CRM activity timeline.

### PATCH /api/meetings/:id/cancel
**Auth:** Required | **Roles:** Participant
**Description:** Cancel a meeting.
**Request Body:**
```json
{ "reason": "Schedule conflict" }
```
**Response:**
```json
{
  "success": true,
  "message": "Meeting cancelled"
}
```
**Notes:** N/A

### PATCH /api/meetings/:id/no-show
**Auth:** Required | **Roles:** Participant
**Description:** Mark meeting as no-show.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "message": "Marked as no-show"
}
```
**Notes:** Used for CRM reporting.


## 9. CRM & Leads (`/api/leads/*`, `/api/opportunities/*`)

### GET /api/leads
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Get list of leads.
**Request Body/Query:**
```json
{ "stage": "new", "source": "inbound" }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "l_1", "name": "Acme Corp", "score": 90 } ]
}
```
**Notes:** N/A

### POST /api/leads
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Create lead manually.
**Request Body:**
```json
{
  "company_name": "Globex",
  "contact_name": "Hank Scorpio",
  "email": "hank@globex.com"
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "l_2" }
}
```
**Notes:** N/A

### GET /api/leads/:id
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Get lead detail.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "l_1", "score": 90, "stage": "new" }
}
```
**Notes:** N/A

### PUT /api/leads/:id
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Update lead information.
**Request Body:**
```json
{ "notes": "Highly interested" }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "l_1" }
}
```
**Notes:** N/A

### PATCH /api/leads/:id/stage
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Move lead to a new stage.
**Request Body:**
```json
{ "stage": "contacted" }
```
**Response:**
```json
{
  "success": true,
  "message": "Stage updated"
}
```
**Notes:** N/A

### PATCH /api/leads/:id/qualify
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Update BANT qualification data.
**Request Body:**
```json
{
  "budget": "10k-50k",
  "authority": "decision_maker",
  "need": "high",
  "timeline": "3_months"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Qualification updated"
}
```
**Notes:** Impacts lead score.

### GET /api/leads/pipeline
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Pipeline summary.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": {
    "new": 10,
    "contacted": 5,
    "qualified": 2
  }
}
```
**Notes:** N/A

### GET /api/leads/overdue
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Leads with overdue follow-ups.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "l_1", "last_contact": "2023-12-01" } ]
}
```
**Notes:** N/A

### POST /api/opportunities
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Create opportunity from a lead.
**Request Body:**
```json
{
  "lead_id": "l_1",
  "value": 25000,
  "expected_close_date": "2024-06-01"
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "opp_1" }
}
```
**Notes:** Marks lead as converted.

### GET /api/opportunities/:id
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Get opportunity detail.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "opp_1", "value": 25000 }
}
```
**Notes:** N/A

### PUT /api/opportunities/:id
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Update opportunity.
**Request Body:**
```json
{ "probability": 80 }
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "opp_1", "probability": 80 }
}
```
**Notes:** N/A

### POST /api/proposals
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Create a proposal for an opportunity.
**Request Body:**
```json
{
  "opportunity_id": "opp_1",
  "document_url": "https://url.to/proposal.pdf",
  "total_amount": 22000
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "prop_1" }
}
```
**Notes:** N/A

### GET /api/proposals/:id
**Auth:** Required | **Roles:** Any (if access link shared)
**Description:** Get proposal detail.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "prop_1", "status": "draft" }
}
```
**Notes:** N/A

### PATCH /api/proposals/:id/send
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Send proposal to customer via email.
**Request Body:**
```json
{ "email": "client@example.com" }
```
**Response:**
```json
{
  "success": true,
  "message": "Proposal sent"
}
```
**Notes:** Updates status to 'sent'.

### POST /api/activities
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Log a CRM activity (call, email, note).
**Request Body:**
```json
{
  "entity_type": "lead",
  "entity_id": "l_1",
  "type": "call",
  "notes": "Discussed pricing."
}
```
**Response:**
```json
{
  "success": true,
  "data": { "id": "act_1" }
}
```
**Notes:** N/A

### GET /api/activities
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Get activities for a specific entity.
**Request Body/Query:**
```json
{ "entity_type": "lead", "entity_id": "l_1" }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "act_1", "type": "call" } ]
}
```
**Notes:** N/A


## 10. AI Services (`/api/ai/*`)

### POST /api/ai/research
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Generate market research report on a company.
**Request Body:**
```json
{ "target_company": "Stripe", "focus_areas": ["pricing", "competitors"] }
```
**Response:**
```json
{
  "success": true,
  "data": { "report_text": "Stripe is a major player in..." }
}
```
**Notes:** Wraps LLM calls.

### POST /api/ai/icp
**Auth:** Required | **Roles:** company_admin
**Description:** Generate Ideal Customer Profile based on company data.
**Request Body:**
```json
{ "company_id": "c_1" }
```
**Response:**
```json
{
  "success": true,
  "data": { "icp_description": "Mid-market tech companies..." }
}
```
**Notes:** N/A

### POST /api/ai/match
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Find matching businesses for a product using AI vector search.
**Request Body:**
```json
{ "product_id": "p_1", "limit": 5 }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "company_id": "c_99", "match_score": 0.95 } ]
}
```
**Notes:** Relies on embeddings.

### POST /api/ai/score
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** AI-driven scoring for a lead.
**Request Body:**
```json
{ "lead_id": "l_1" }
```
**Response:**
```json
{
  "success": true,
  "data": { "score": 88, "reasons": ["High budget", "Decision maker"] }
}
```
**Notes:** Updates lead score in DB.

### POST /api/ai/outreach-draft
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Generate personalized outreach email/message.
**Request Body:**
```json
{ "lead_id": "l_1", "tone": "professional" }
```
**Response:**
```json
{
  "success": true,
  "data": { "draft": "Subject: Improving your workflow...\nHi Hank," }
}
```
**Notes:** N/A

### POST /api/ai/ask
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** RAG query over company documents (proposals, meeting notes).
**Request Body:**
```json
{ "query": "What did we quote Acme Corp last year?" }
```
**Response:**
```json
{
  "success": true,
  "data": { "answer": "You quoted Acme Corp $15,000 on Oct 12, 2023." }
}
```
**Notes:** Uses vector DB and LLM.

### POST /api/ai/summarize-notes
**Auth:** Required | **Roles:** company_admin, salesperson
**Description:** Summarize raw meeting notes into CRM fields.
**Request Body:**
```json
{ "raw_notes": "Client wants to move forward next month. Budget is 20k." }
```
**Response:**
```json
{
  "success": true,
  "data": { "summary": "Positive meeting", "extracted_budget": "20k", "timeline": "1 month" }
}
```
**Notes:** N/A


## 11. File Upload (`/api/upload/*`)

### POST /api/upload/image
**Auth:** Required | **Roles:** Any
**Description:** Upload an image file (avatar, logo, post image).
**Request Body:**
Multipart form-data: `file=image.png`
**Query:** `?context=avatar`
**Response:**
```json
{
  "success": true,
  "data": { "url": "https://bucket.aws.com/images/123.png" }
}
```
**Notes:** Compresses and validates image type.

### POST /api/upload/document
**Auth:** Required | **Roles:** Any
**Description:** Upload PDF or document (proposals, contracts).
**Request Body:**
Multipart form-data: `file=doc.pdf`
**Response:**
```json
{
  "success": true,
  "data": { "url": "https://bucket.aws.com/docs/456.pdf" }
}
```
**Notes:** N/A

### POST /api/upload/bulk
**Auth:** Required | **Roles:** company_admin
**Description:** Bulk CSV/XLSX import for leads/products.
**Request Body:**
Multipart form-data: `file=leads.csv`
**Response:**
```json
{
  "success": true,
  "message": "Import job started",
  "job_id": "job_123"
}
```
**Notes:** Processed asynchronously.


## 12. Admin (`/api/admin/*`)

### GET /api/admin/users
**Auth:** Required | **Roles:** platform_admin
**Description:** List all platform users.
**Request Body/Query:**
```json
{ "page": 1, "search": "john" }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "u_1", "email": "john@example.com" } ]
}
```
**Notes:** N/A

### PATCH /api/admin/users/:id/status
**Auth:** Required | **Roles:** platform_admin
**Description:** Activate or deactivate user.
**Request Body:**
```json
{ "status": "suspended", "reason": "TOS Violation" }
```
**Response:**
```json
{
  "success": true,
  "message": "User status updated"
}
```
**Notes:** N/A

### GET /api/admin/companies
**Auth:** Required | **Roles:** platform_admin
**Description:** List all registered companies.
**Request Body/Query:**
```json
{ "verified": false }
```
**Response:**
```json
{
  "success": true,
  "data": [ { "id": "c_1", "name": "Shady Corp" } ]
}
```
**Notes:** N/A

### PATCH /api/admin/companies/:id/verify
**Auth:** Required | **Roles:** platform_admin
**Description:** Verify or reject a company application.
**Request Body:**
```json
{ "verified": true }
```
**Response:**
```json
{
  "success": true,
  "message": "Company verified"
}
```
**Notes:** Grants company platform visibility.

### GET /api/admin/stats
**Auth:** Required | **Roles:** platform_admin
**Description:** Platform-wide statistics.
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": {
    "total_users": 5000,
    "total_companies": 1200,
    "active_users_24h": 450
  }
}
```
**Notes:** N/A

### GET /api/admin/moderation
**Auth:** Required | **Roles:** platform_admin
**Description:** Queue of flagged content (posts, comments).
**Request Body/Query:**
```json
{}
```
**Response:**
```json
{
  "success": true,
  "data": [ { "type": "post", "id": "post_bad", "flag_count": 3 } ]
}
```
**Notes:** N/A


## 13. WebSocket Events (Socket.IO)

**Connection:** Client connects and emits `authenticate` with JWT token payload.

### Client → Server Events:
- `message:send`: `{ "conversation_id": "c_1", "content": "Hi" }`
- `typing:start`: `{ "conversation_id": "c_1" }`
- `typing:stop`: `{ "conversation_id": "c_1" }`
- `conversation:read`: `{ "conversation_id": "c_1", "message_id": "m_5" }`

### Server → Client Events:
- `message:new`: `{ "id": "m_6", "conversation_id": "c_1", "content": "Hi", "sender_id": "u_2" }`
- `typing:indicator`: `{ "conversation_id": "c_1", "user_id": "u_2", "is_typing": true }`
- `notification:new`: `{ "id": "n_1", "type": "meeting_request", "message": "New request" }`
- `meeting:update`: `{ "meeting_id": "m_1", "status": "accepted" }`
