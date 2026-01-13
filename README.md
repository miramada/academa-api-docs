# 📱 Academa Dashboard - React Native Integration Guide

**Version:** 1.3.0  
**Last Updated:** January 14, 2025  
**Base URL:** `https://www.hurras.net/wp-json`

---

## 📋 Table of Contents

1. [Quick Start](#quick-start)
2. [Authentication System](#authentication-system)
3. [API Endpoints Reference](#api-endpoints-reference)
4. [React Native Implementation](#react-native-implementation)
5. [Error Handling](#error-handling)
6. [Best Practices](#best-practices)

---

## 🚀 Quick Start

### Required NPM Packages

```bash
npm install @react-native-async-storage/async-storage
npm install axios  # Optional, but recommended
```

### Test Credentials

```javascript
const TEST_USER = {
  username: 'trial',
  password: '1551981'
};
```

### Quick Test

```javascript
const response = await fetch('https://www.hurras.net/wp-json/jwt-auth/v1/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    username: 'trial',
    password: '1551981'
  })
});

const data = await response.json();
console.log('Access Token:', data.access_token);
```

---

## 🔐 Authentication System

### Overview

The API uses **JWT (JSON Web Tokens)** for authentication. There are two types of tokens:

- **Access Token**: Short-lived (1 hour) - used for API calls
- **Refresh Token**: Long-lived (30 days) - used to generate new access tokens

### Authentication Methods

**Method 1: Custom Header (✅ Recommended)**
```javascript
headers: {
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Method 2: Query Parameter (Testing Only)**
```
?jwt_token=YOUR_ACCESS_TOKEN
```

**⚠️ DO NOT USE:** `Authorization: Bearer TOKEN` - This has Apache compatibility issues.

---

## 📡 API Endpoints Reference

### 1. Authentication Endpoints

#### 1.1 Login (Generate Tokens)

**Endpoint:** `POST /jwt-auth/v1/token`

**Request:**
```javascript
{
  "username": "trial",
  "password": "1551981"
}
```

**Success Response (200):**
```javascript
{
  "access_token": "eyJ0eXAiOiJKV1Qi...",
  "refresh_token": "eyJ0eXAiOiJKV1Qi...",
  "token_type": "Bearer",
  "expires_in": 3600,        // seconds (1 hour)
  "user_id": 89,
  "user_email": "user@example.com",
  "user_nicename": "trial",
  "user_display_name": "Trial"
}
```

**Error Response (403):**
```javascript
{
  "code": "invalid_credentials",
  "message": "Invalid username or password",
  "data": { "status": 403 }
}
```

---

#### 1.2 Refresh Token

**Endpoint:** `POST /jwt-auth/v1/token/refresh`

**Request:**
```javascript
{
  "refresh_token": "eyJ0eXAiOiJKV1Qi..."
}
```

**Success Response (200):**
```javascript
{
  "access_token": "NEW_ACCESS_TOKEN",
  "refresh_token": "NEW_REFRESH_TOKEN",  // Also rotated for security
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": 89
}
```

**Usage Note:** Always save BOTH tokens when refreshing.

---

#### 1.3 Validate Token

**Endpoint:** `POST /jwt-auth/v1/token/validate`

**Request:**
```javascript
{
  "token": "eyJ0eXAiOiJKV1Qi..."
}
```

**Success Response (200):**
```javascript
{
  "code": "jwt_valid",
  "data": {
    "status": 200,
    "user_id": 89,
    "token_type": "access"
  }
}
```

**Error Response (403):**
```javascript
{
  "code": "jwt_expired",
  "message": "Token expired",
  "data": { "status": 403 }
}
```

---

### 2. Dashboard Endpoints

#### 2.1 Dashboard Overview 🏠

**Endpoint:** `GET /academa-dashboard/v1/user/dashboard-overview`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "user_info": {
    "id": 89,
    "display_name": "Trial",
    "first_name": "تجربة",
    "last_name": "تجربة",
    "email": "mirama@gmail.com",
    "avatar_url": "https://secure.gravatar.com/avatar/...",
    "academic_specialization": "",
    "birth_date": "",
    "gender": "",
    "stats": {
      "total_courses_enrolled": 6,
      "total_courses_completed": 0,
      "total_certificates_earned": 0
    }
  },
  "current_stage": {
    "id": 114862,
    "name": "2026: المرحلة التأسيسية | الدفعة الرابعة",
    "welcome_message_html": "<p>مرحبًا، <strong>Trial</strong>...</p>"
  },
  "latest_updates": [
    {
      "id": 227914,
      "title": "تابع جدولك اليومي",
      "content_html": "<p>محتوى الإعلان...</p>",
      "link_url": "https://example.com",
      "link_text": "الذهاب للموقع"
    }
  ],
  "stage_courses": [
    {
      "id": 151741,
      "title": "2026: الصلب والفداء",
      "permalink": "https://www.hurras.net/courses/...",
      "progress_percentage": 0,
      "status": "not-started",  // or "in-progress", "completed"
      "start_date_timestamp": null,
      "has_certificate_earned": false,
      "certificate_url": null,
      "number_of_quizzes": 0,
      "quizzes_completed_count": 0,
      "quizzes_passed_count": 0
    }
  ],
  "special_enrolled_courses": [],
  "logout_url": "https://www.hurras.net/wp-login.php?action=logout..."
}
```

**Use Cases:**
- Display on home screen
- Show user stats
- List enrolled courses
- Display announcements

---

#### 2.2 Update User Profile

**Endpoint:** `POST /academa-dashboard/v1/user/me/update-profile`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN',
  'Content-Type': 'application/json'
}
```

**Request Body:**
```javascript
{
  "first_name": "أحمد",
  "last_name": "محمد",
  "email": "ahmed@example.com",
  "academic_specialization": "Computer Science",
  "birth_date": "1995-05-15",  // YYYY-MM-DD format
  "gender": "male"             // "male", "female", or "other"
}
```

**Success Response (200):**
```javascript
{
  "success": true,
  "message": "Profile updated successfully",
  "user": {
    "id": 89,
    "display_name": "أحمد محمد",
    "email": "ahmed@example.com"
  }
}
```

---

### 3. Learning Management System (LMS) Endpoints

#### 3.1 Get User Courses

**Endpoint:** `GET /academa-dashboard/v1/lms/courses`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "provider": "learndash",
  "courses": [
    {
      "id": 151741,
      "title": "Course Title",
      "link": "https://www.hurras.net/courses/...",
      "thumbnail": "https://www.hurras.net/wp-content/uploads/...",
      "progress": 25,          // 0-100
      "status": "in-progress"  // "not-started", "in-progress", "completed"
    }
  ]
}
```

---

#### 3.2 Get User Certificates

**Endpoint:** `GET /academa-dashboard/v1/lms/certificates`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "success": true,
  "provider": "auto",
  "certificates": [
    {
      "certificate_id": 12345,
      "course_name": "Course Name",
      "earned_date": "2024-01-15",
      "certificate_url": "https://www.hurras.net/certificates/..."
    }
  ],
  "performance": []
}
```

---

#### 3.3 Get Announcements

**Endpoint:** `GET /academa-dashboard/v1/announcements`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "announcements": [
    {
      "id": 227914,
      "title": "Announcement Title",
      "content_html": "<p>Content...</p>",
      "link_url": "https://example.com",
      "link_text": "Read More"
    }
  ]
}
```

---

### 4. Marketplace Endpoints

#### 4.1 Get User Orders

**Endpoint:** `GET /academa-dashboard/v1/marketplace/orders`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "provider": "edd",  // Easy Digital Downloads
  "orders": [
    {
      "id": 12345,
      "number": "ORDER-12345",
      "date": "2024-01-15",
      "total": "ر.س 99.00",
      "status": "complete",
      "status_label": "Complete",
      "view_url": "https://www.hurras.net/checkout/..."
    }
  ]
}
```

---

#### 4.2 Get User Subscriptions

**Endpoint:** `GET /academa-dashboard/v1/marketplace/subscriptions`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "success": true,
  "provider": "edd",
  "subscriptions": [
    {
      "id": 67890,
      "product_name": "Premium Membership",
      "status": "active",
      "status_label": "Active",
      "renews_at": "2024-02-15",
      "amount": "ر.س 99.00 / month",
      "payment_method": "PayPal",
      "actions": {
        "cancel_url": "https://www.hurras.net/cancel/...",
        "renew_url": "https://www.hurras.net/renew/...",
        "update_url": "https://www.hurras.net/payment-methods/...",
        "view_url": "https://www.hurras.net/subscriptions/..."
      }
    }
  ]
}
```

---

#### 4.3 Get User Downloads

**Endpoint:** `GET /academa-dashboard/v1/marketplace/downloads`

**Headers:**
```javascript
{
  'X-JWT-Token': 'YOUR_ACCESS_TOKEN'
}
```

**Success Response (200):**
```javascript
{
  "provider": "edd",
  "downloads": [
    {
      "id": "12345-151741-0",
      "title": "Premium Course Materials",
      "download_url": "https://www.hurras.net/download/...",
      "version": "1.2.0",
      "changelog_url": "",
      "license_status": "active",
      "is_expired": false,
      "date": "2024-01-15"
    }
  ]
}
```

---

## 💻 React Native Implementation

### Complete Service Layer

Create a file: `services/AcademaAPI.js`

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

const API_BASE = 'https://www.hurras.net/wp-json';

// ============================================
// 1. AUTHENTICATION SERVICE
// ============================================

export const AuthService = {
  
  /**
   * Login user and save tokens
   * @param {string} username 
   * @param {string} password 
   * @returns {Promise<{success: boolean, data?: object, error?: string}>}
   */
  login: async (username, password) => {
    try {
      const response = await fetch(`${API_BASE}/jwt-auth/v1/token`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ username, password }),
      });
      
      const data = await response.json();
      
      if (response.ok) {
        // Save tokens securely
        await AsyncStorage.multiSet([
          ['@access_token', data.access_token],
          ['@refresh_token', data.refresh_token],
          ['@user_id', data.user_id.toString()],
          ['@user_email', data.user_email],
          ['@user_display_name', data.user_display_name],
        ]);
        
        return { success: true, data };
      } else {
        return { success: false, error: data.message || 'Login failed' };
      }
    } catch (error) {
      console.error('Login error:', error);
      return { success: false, error: error.message };
    }
  },
  
  /**
   * Logout user and clear tokens
   */
  logout: async () => {
    try {
      await AsyncStorage.multiRemove([
        '@access_token',
        '@refresh_token',
        '@user_id',
        '@user_email',
        '@user_display_name',
      ]);
    } catch (error) {
      console.error('Logout error:', error);
    }
  },
  
  /**
   * Refresh access token
   * @returns {Promise<string|null>} New access token or null
   */
  refreshToken: async () => {
    try {
      const refreshToken = await AsyncStorage.getItem('@refresh_token');
      
      if (!refreshToken) {
        return null;
      }
      
      const response = await fetch(`${API_BASE}/jwt-auth/v1/token/refresh`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ refresh_token: refreshToken }),
      });
      
      const data = await response.json();
      
      if (response.ok) {
        // Save new tokens
        await AsyncStorage.multiSet([
          ['@access_token', data.access_token],
          ['@refresh_token', data.refresh_token],
        ]);
        
        return data.access_token;
      }
      
      return null;
    } catch (error) {
      console.error('Refresh token error:', error);
      return null;
    }
  },
  
  /**
   * Get current access token
   * @returns {Promise<string|null>}
   */
  getToken: async () => {
    try {
      return await AsyncStorage.getItem('@access_token');
    } catch (error) {
      console.error('Get token error:', error);
      return null;
    }
  },
  
  /**
   * Check if user is logged in
   * @returns {Promise<boolean>}
   */
  isLoggedIn: async () => {
    const token = await AuthService.getToken();
    return token !== null;
  },
  
};

// ============================================
// 2. API SERVICE (with auto token refresh)
// ============================================

export const ApiService = {
  
  /**
   * Make API call with automatic token refresh
   * @param {string} endpoint - API endpoint path
   * @param {object} options - Fetch options
   * @returns {Promise<object>}
   */
  call: async (endpoint, options = {}) => {
    let token = await AuthService.getToken();
    
    if (!token) {
      throw new Error('No authentication token available');
    }
    
    try {
      const response = await fetch(`${API_BASE}${endpoint}`, {
        ...options,
        headers: {
          'X-JWT-Token': token,
          'Content-Type': 'application/json',
          ...options.headers,
        },
      });
      
      const data = await response.json();
      
      // If token expired, refresh and retry
      if (response.status === 403 && data.code === 'jwt_expired') {
        console.log('Token expired, refreshing...');
        
        token = await AuthService.refreshToken();
        
        if (token) {
          // Retry with new token
          const retryResponse = await fetch(`${API_BASE}${endpoint}`, {
            ...options,
            headers: {
              'X-JWT-Token': token,
              'Content-Type': 'application/json',
              ...options.headers,
            },
          });
          
          return await retryResponse.json();
        } else {
          // Refresh failed, user needs to login again
          await AuthService.logout();
          throw new Error('Session expired. Please login again.');
        }
      }
      
      if (!response.ok) {
        throw new Error(data.message || 'API request failed');
      }
      
      return data;
      
    } catch (error) {
      console.error('API call error:', error);
      throw error;
    }
  },
  
  /**
   * Get dashboard overview data
   * @returns {Promise<object>}
   */
  getDashboard: async () => {
    return await ApiService.call('/academa-dashboard/v1/user/dashboard-overview');
  },
  
  /**
   * Update user profile
   * @param {object} profileData 
   * @returns {Promise<object>}
   */
  updateProfile: async (profileData) => {
    return await ApiService.call('/academa-dashboard/v1/user/me/update-profile', {
      method: 'POST',
      body: JSON.stringify(profileData),
    });
  },
  
  /**
   * Get user courses
   * @returns {Promise<object>}
   */
  getCourses: async () => {
    return await ApiService.call('/academa-dashboard/v1/lms/courses');
  },
  
  /**
   * Get user certificates
   * @returns {Promise<object>}
   */
  getCertificates: async () => {
    return await ApiService.call('/academa-dashboard/v1/lms/certificates');
  },
  
  /**
   * Get user orders
   * @returns {Promise<object>}
   */
  getOrders: async () => {
    return await ApiService.call('/academa-dashboard/v1/marketplace/orders');
  },
  
  /**
   * Get user subscriptions
   * @returns {Promise<object>}
   */
  getSubscriptions: async () => {
    return await ApiService.call('/academa-dashboard/v1/marketplace/subscriptions');
  },
  
  /**
   * Get user downloads
   * @returns {Promise<object>}
   */
  getDownloads: async () => {
    return await ApiService.call('/academa-dashboard/v1/marketplace/downloads');
  },
  
  /**
   * Get announcements
   * @returns {Promise<object>}
   */
  getAnnouncements: async () => {
    return await ApiService.call('/academa-dashboard/v1/announcements');
  },
  
};
```

---

### Example Screen: Dashboard

Create file: `screens/DashboardScreen.js`

```javascript
import React, { useEffect, useState } from 'react';
import {
  View,
  Text,
  ScrollView,
  ActivityIndicator,
  RefreshControl,
  StyleSheet,
  Alert,
} from 'react-native';
import { ApiService, AuthService } from '../services/AcademaAPI';

export default function DashboardScreen({ navigation }) {
  const [dashboard, setDashboard] = useState(null);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  
  useEffect(() => {
    loadDashboard();
  }, []);
  
  const loadDashboard = async () => {
    try {
      setLoading(true);
      const data = await ApiService.getDashboard();
      setDashboard(data);
    } catch (error) {
      console.error('Dashboard error:', error);
      
      if (error.message.includes('Session expired')) {
        // Redirect to login
        Alert.alert(
          'Session Expired',
          'Please login again',
          [{ text: 'OK', onPress: () => navigation.replace('Login') }]
        );
      } else {
        Alert.alert('Error', error.message);
      }
    } finally {
      setLoading(false);
    }
  };
  
  const onRefresh = async () => {
    setRefreshing(true);
    await loadDashboard();
    setRefreshing(false);
  };
  
  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading dashboard...</Text>
      </View>
    );
  }
  
  if (!dashboard) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>Failed to load dashboard</Text>
      </View>
    );
  }
  
  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      {/* User Info */}
      <View style={styles.userCard}>
        <Text style={styles.welcomeText}>
          مرحباً، {dashboard.user_info.display_name}!
        </Text>
        <View style={styles.statsRow}>
          <View style={styles.statItem}>
            <Text style={styles.statValue}>
              {dashboard.user_info.stats.total_courses_enrolled}
            </Text>
            <Text style={styles.statLabel}>Courses</Text>
          </View>
          <View style={styles.statItem}>
            <Text style={styles.statValue}>
              {dashboard.user_info.stats.total_courses_completed}
            </Text>
            <Text style={styles.statLabel}>Completed</Text>
          </View>
          <View style={styles.statItem}>
            <Text style={styles.statValue}>
              {dashboard.user_info.stats.total_certificates_earned}
            </Text>
            <Text style={styles.statLabel}>Certificates</Text>
          </View>
        </View>
      </View>
      
      {/* Current Stage */}
      {dashboard.current_stage.id && (
        <View style={styles.stageCard}>
          <Text style={styles.sectionTitle}>مرحلتك الحالية</Text>
          <Text style={styles.stageName}>{dashboard.current_stage.name}</Text>
        </View>
      )}
      
      {/* Announcements */}
      {dashboard.latest_updates.length > 0 && (
        <View style={styles.announcementsSection}>
          <Text style={styles.sectionTitle}>آخر التحديثات</Text>
          {dashboard.latest_updates.map((announcement) => (
            <View key={announcement.id} style={styles.announcementCard}>
              <Text style={styles.announcementTitle}>
                {announcement.title}
              </Text>
            </View>
          ))}
        </View>
      )}
      
      {/* Courses */}
      {dashboard.stage_courses.length > 0 && (
        <View style={styles.coursesSection}>
          <Text style={styles.sectionTitle}>المساقات</Text>
          {dashboard.stage_courses.map((course) => (
            <View key={course.id} style={styles.courseCard}>
              <Text style={styles.courseTitle}>{course.title}</Text>
              <View style={styles.progressBar}>
                <View
                  style={[
                    styles.progressFill,
                    { width: `${course.progress_percentage}%` },
                  ]}
                />
              </View>
              <Text style={styles.progressText}>
                {course.progress_percentage}% Complete
              </Text>
            </View>
          ))}
        </View>
      )}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F5F5F5',
  },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5F5F5',
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorText: {
    fontSize: 16,
    color: '#FF3B30',
  },
  userCard: {
    backgroundColor: '#FFF',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  welcomeText: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 15,
    textAlign: 'right',
  },
  statsRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  statItem: {
    alignItems: 'center',
  },
  statValue: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  statLabel: {
    fontSize: 14,
    color: '#666',
    marginTop: 5,
  },
  stageCard: {
    backgroundColor: '#FFF',
    marginHorizontal: 15,
    marginBottom: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'right',
  },
  stageName: {
    fontSize: 16,
    color: '#333',
    textAlign: 'right',
  },
  announcementsSection: {
    marginHorizontal: 15,
    marginBottom: 15,
  },
  announcementCard: {
    backgroundColor: '#FFF',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  announcementTitle: {
    fontSize: 16,
    fontWeight: '600',
    textAlign: 'right',
  },
  coursesSection: {
    marginHorizontal: 15,
    marginBottom: 15,
  },
  courseCard: {
    backgroundColor: '#FFF',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  courseTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 10,
    textAlign: 'right',
  },
  progressBar: {
    height: 8,
    backgroundColor: '#E0E0E0',
    borderRadius: 4,
    overflow: 'hidden',
  },
  progressFill: {
    height: '100%',
    backgroundColor: '#4CAF50',
  },
  progressText: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
  },
});
```

---

## ⚠️ Error Handling

### Common Error Codes

| Code | Status | Description | Solution |
|------|--------|-------------|----------|
| `jwt_expired` | 403 | Access token expired | Use refresh token |
| `jwt_invalid` | 403 | Invalid token format | Re-login required |
| `rest_forbidden` | 401 | No auth token provided | Send token in header |
| `invalid_credentials` | 403 | Wrong username/password | Check credentials |
| `too_many_requests` | 429 | Rate limit exceeded | Wait 15 minutes |

### Error Handling Example

```javascript
try {
  const data = await ApiService.getDashboard();
  // Success
} catch (error) {
  if (error.message.includes('Session expired')) {
    // Redirect to login
    navigation.replace('Login');
  } else if (error.message.includes('Network')) {
    // Show offline message
    Alert.alert('No Internet', 'Please check your connection');
  } else {
    // Generic error
    Alert.alert('Error', error.message);
  }
}
```

---

## ✅ Best Practices

### 1. Token Management

```javascript
// Always check if logged in before API calls
const isLoggedIn = await AuthService.isLoggedIn();
if (!isLoggedIn) {
  navigation.replace('Login');
  return;
}

// Let ApiService handle token refresh automatically
const data = await ApiService.getDashboard();
```

### 2. Secure Storage

```javascript
// ✅ Use AsyncStorage for tokens
import AsyncStorage from '@react-native-async-storage/async-storage';

// ❌ DON'T store in plain state
const [token, setToken] = useState(''); // NOT secure
```

### 3. Loading States

```javascript
const [loading, setLoading] = useState(true);
const [data, setData] = useState(null);

useEffect(() => {
  loadData();
}, []);

const loadData = async () => {
  setLoading(true);
  try {
    const result = await ApiService.getDashboard();
    setData(result);
  } catch (error) {
    console.error(error);
  } finally {
    setLoading(false);
  }
};
```

### 4. Pull-to-Refresh

```javascript
const [refreshing, setRefreshing] = useState(false);

const onRefresh = async () => {
  setRefreshing(true);
  await loadData();
  setRefreshing(false);
};

// In ScrollView:
<ScrollView
  refreshControl={
    <RefreshControl 
      refreshing={refreshing} 
      onRefresh={onRefresh} 
    />
  }
>
```

### 5. Offline Support (Optional)

```javascript
import NetInfo from '@react-native-community/netinfo';

// Check connectivity before API calls
const isConnected = await NetInfo.fetch().then(state => state.isConnected);

if (!isConnected) {
  Alert.alert('No Internet', 'Please check your connection');
  return;
}
```

---

## 🔒 Security Notes

1. **HTTPS Only**: All API calls use HTTPS (`https://www.hurras.net`)
2. **Token Storage**: Use `AsyncStorage` (secure on both iOS/Android)
3. **Token Rotation**: Refresh tokens are rotated on each refresh
4. **Rate Limiting**: Max 5 failed login attempts per 15 minutes
5. **Token Expiration**: 
   - Access Token: 1 hour
   - Refresh Token: 30 days

---

## 📝 Changelog

### Version 1.3.0 (January 14, 2025)
- ✅ Fixed Apache header compatibility issues
- ✅ Added `X-JWT-Token` custom header support
- ✅ Improved token refresh mechanism
- ✅ Added comprehensive error handling

### Version 1.2.0 (January 2025)
- Initial React Native documentation release

---

**End of Documentation**
