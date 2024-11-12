# Uplas-MVP
#MVP for Uplas AI


// App.js
import React, { useState } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Layout from './components/Layout';
import Home from './pages/Home';
import CourseDetails from './pages/CourseDetails';
import Login from './pages/Login';
import Signup from './pages/Signup';

function App() {
  const [darkMode, setDarkMode] = useState(false);

  const toggleDarkMode = () => {
    setDarkMode(!darkMode);
  };

  return (
    <Router>
      <Layout darkMode={darkMode} toggleDarkMode={toggleDarkMode}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/courses/:courseId" element={<CourseDetails />} />
          <Route path="/login" element={<Login />} />
          <Route path="/signup" element={<Signup />} />
        </Routes>
      </Layout>
    </Router>
  );
}

export default App;

// CourseDetails.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function CourseDetails({ match }) {
  const [course, setCourse] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchCourse = async () => {
      try {
        const response = await axios.get(`/api/courses/${match.params.courseId}`);
        setCourse(response.data);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchCourse();
  }, [match.params.courseId]);

  return (
    <div>
      {isLoading ? (
        <p>Loading course details...</p>
      ) : error ? (
        <p>Error fetching course details: {error.message}</p>
      ) : (
        <div>
          <h2>{course.title}</h2>
          <p>{course.description}</p>
          {/* Add more course details here */}
        </div>
      )}
    </div>
  );
}

export default CourseDetails;

// Notification.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function Notification() {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const fetchNotifications = async () => {
      try {
        const response = await axios.get('/api/notifications');
        setNotifications(response.data);
      } catch (error) {
        console.error('Error fetching notifications:', error);
      }
    };

    fetchNotifications();
  }, []);

  return (
    <div>
      {/* Display notifications here */}
      {notifications.map((notification) => (
        <div key={notification.id}>{notification.message}</div>
      ))}
    </div>
  );
}

export default Notification;

// Login.js and Signup.js (Implement user authentication logic)
// ...

// Layout.js (Add notification icon and handle click)
// ...

// Global Styles (styles/global.css)
// ... (Add Bootstrap or custom CSS for responsive design)


# Database Setup (MySQL):

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'your_username',  # Replace with your MySQL database name
        'USER': 'your_username',  # Replace with your MySQL username
        'PASSWORD': 'your_password',  # Replace with your MySQL password
        'HOST': 'your_host',       # Replace with your MySQL host (localhost or server IP)
        'PORT': '3306',
    }
}


# Models (App/models.py):


from django.db import models
from django.contrib.auth.hashers import make_password  # For password hashing

class Course(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    instructor = models.CharField(max_length=100, blank=True)  # Optional instructor field
    duration = models.CharField(max_length=50, blank=True)  # Optional duration field
    price = models.DecimalField(max_digits=10, decimal_places=2, blank=True, null=True)  # Optional price field
    image_url = models.URLField(blank=True)  # Optional image URL field

class User(models.Model):
    username = models.CharField(max_length=50, unique=True)
    email = models.EmailField(unique=True)
    password = models.CharField(max_length=128)  # Hashed password field

    def set_password(self, plain_password):
        self.password = make_password(plain_password)

    # Add more fields like first_name, last_name, and role
    first_name = models.CharField(max_length=50, blank=True)
    last_name = models.CharField(max_length=50, blank=True)
    role = models.CharField(max_length=20, choices=(('student', 'Student'), ('instructor', 'Instructor')), default='student')

class Notification(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    message = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)


# Views (app/views.py):


from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required

def user_login(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(username=username, password=password)
        if user is not None:
            login(request, user)
            # Redirect to homepage or another page after successful login
            return redirect('home')
        else:
            # Handle invalid login attempt (e.g., display an error message)
            pass
    return render(request, 'login.html')  # Replace with your login template

@login_required
def user_signup(request):
    if request.method == 'POST':
        username = request.POST['username']
        email = request.POST['email']
        password = request.POST['password']
        first_name = request.POST['first_name']
        last_name = request.POST['last_name']
        role = request.POST['role']

        # Create user with hashed password
        user = User.objects.create(username=username, email=email, first_name=first_name, last_name=last_name, role=role)
        user.set_password(password)
        user.save()

        # Handle successful signup (e.g., redirect to login page)
        return redirect('login')
    return render(request, 'signup.html')  # Replace with your signup template


# URLs (app/urls.py):


from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('courses/<int:course_id>/', views.course_details, name='course_details'),
    path('login/', views.user_login, name='login'),
    path('signup/', views.user_signup, name='signup'),
    path('notifications/', views.get_notifications, name='notifications'),
]
