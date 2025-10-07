# AuthenticationService Documentation

## 📋 **Tổng quan**

`AuthenticationService` là core service để handle user authentication, session management, token refresh, và security features. Service này manage toàn bộ authentication lifecycle từ login đến logout.

**Location**: `src/app/services/auth/authentication.service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Features**

### **Authentication State Management**
```typescript
interface AuthState {
    isAuthenticated: boolean;
    isLoading: boolean;
    user?: any;
    token?: TokenResponse;
    error?: string;
    lastActivity?: Date;
}
```

### **Core Methods**
```typescript
login(credentials: LoginCredentials): Promise<AuthResult>
logout(): Promise<void>
refreshToken(): Promise<TokenResponse>
validateToken(token: string): boolean
isAuthenticated(): boolean
getCurrentToken(): string | null
getAuthState(): Observable<AuthState>
```

---

## 🚀 **Basic Usage**

### **User Login**
```typescript
export class LoginPage extends PageBase {
    loginForm = {
        username: '',
        password: ''
    };
    
    constructor(private authService: AuthenticationService) {
        super();
    }
    
    async login() {
        try {
            if (!this.loginForm.username || !this.loginForm.password) {
                this.env.showMessage('Please enter username and password', 'warning');
                return;
            }
            
            const credentials: LoginCredentials = {
                username: this.loginForm.username,
                password: this.loginForm.password
            };
            
            const result = await this.authService.login(credentials);
            
            if (result.success) {
                this.env.showMessage('Login successful', 'success');
                this.nav('/dashboard', 'forward');
            } else if (result.requiresMfa) {
                // Handle MFA requirement
                this.showMFADialog();
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    async logout() {
        try {
            await this.authService.logout();
            this.env.showMessage('Logged out successfully', 'success');
            this.nav('/login', 'root');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Authentication State Monitoring**
```typescript
export class AppComponent implements OnInit {
    isAuthenticated = false;
    isLoading = false;
    
    constructor(private authService: AuthenticationService) {}
    
    ngOnInit() {
        // Subscribe to authentication state changes
        this.authService.getAuthState().subscribe(state => {
            this.isAuthenticated = state.isAuthenticated;
            this.isLoading = state.isLoading;
            
            if (state.error) {
                this.env.showMessage(state.error, 'danger');
            }
            
            // Handle authentication state changes
            if (!state.isAuthenticated && this.router.url !== '/login') {
                this.router.navigate(['/login']);
            }
        });
    }
}
```

---

## 🎨 **Advanced Usage**

### **Multi-Factor Authentication (MFA)**
```typescript
export class MFALoginPage extends PageBase {
    mfaCode = '';
    mfaChallenge: any;
    
    constructor(private authService: AuthenticationService) {
        super();
    }
    
    async loginWithMFA() {
        try {
            const credentials: LoginCredentials = {
                username: this.username,
                password: this.password
            };
            
            const result = await this.authService.loginWithMFA(credentials, this.mfaCode);
            
            if (result.success) {
                this.env.showMessage('Login successful', 'success');
                this.nav('/dashboard', 'forward');
            } else {
                this.env.showMessage('Invalid MFA code', 'danger');
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Session Management**
```typescript
export class SessionManagerService {
    private sessionTimeout = 8 * 60 * 60 * 1000; // 8 hours
    private warningTime = 5 * 60 * 1000; // 5 minutes before expiry
    
    constructor(private authService: AuthenticationService) {
        this.setupSessionMonitoring();
    }
    
    private setupSessionMonitoring() {
        // Monitor authentication state
        this.authService.getAuthState().subscribe(state => {
            if (state.isAuthenticated && state.lastActivity) {
                this.checkSessionExpiry(state.lastActivity);
            }
        });
        
        // Listen for session events
        this.env.getEvents().subscribe(event => {
            switch (event.Code) {
                case EVENT_TYPE.USER.SESSION_EXPIRED:
                    this.handleSessionExpired();
                    break;
                case EVENT_TYPE.USER.SESSION_WARNING:
                    this.showSessionWarning();
                    break;
            }
        });
    }
    
    private checkSessionExpiry(lastActivity: Date) {
        const now = new Date();
        const timeSinceActivity = now.getTime() - lastActivity.getTime();
        
        if (timeSinceActivity > this.sessionTimeout) {
            this.env.publishEvent({ Code: EVENT_TYPE.USER.SESSION_EXPIRED });
        } else if (timeSinceActivity > (this.sessionTimeout - this.warningTime)) {
            this.env.publishEvent({ Code: EVENT_TYPE.USER.SESSION_WARNING });
        }
    }
    
    private async handleSessionExpired() {
        await this.env.showAlert(
            'Your session has expired. Please login again.',
            'Session Expired',
            'Session Timeout'
        );
        
        await this.authService.logout();
        this.router.navigate(['/login']);
    }
    
    private async showSessionWarning() {
        const result = await this.env.showPrompt(
            'Your session will expire in 5 minutes. Do you want to extend it?',
            'Session Warning',
            'Session Timeout',
            'Extend Session',
            'Logout'
        );
        
        if (result) {
            // Extend session by making an API call
            await this.extendSession();
        } else {
            await this.authService.logout();
        }
    }
    
    private async extendSession() {
        try {
            // Make a simple API call to extend session
            await this.commonService.connect('GET', 'api/ping').toPromise();
            this.env.showMessage('Session extended', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Token Management**
```typescript
export class TokenManagerService {
    constructor(private authService: AuthenticationService) {
        this.setupTokenRefresh();
    }
    
    private setupTokenRefresh() {
        // Monitor token expiry and refresh automatically
        setInterval(async () => {
            if (this.authService.isAuthenticated()) {
                const token = this.authService.getCurrentToken();
                
                if (token && this.isTokenNearExpiry(token)) {
                    try {
                        await this.authService.refreshToken();
                        console.log('Token refreshed successfully');
                    } catch (error) {
                        console.error('Token refresh failed:', error);
                        await this.authService.logout();
                    }
                }
            }
        }, 60000); // Check every minute
    }
    
    private isTokenNearExpiry(token: string): boolean {
        try {
            const payload = this.parseJWT(token);
            const exp = payload.exp * 1000; // Convert to milliseconds
            const now = Date.now();
            const fiveMinutes = 5 * 60 * 1000;
            
            return (exp - now) < fiveMinutes;
        } catch (error) {
            return true; // If can't parse, assume expired
        }
    }
    
    private parseJWT(token: string): any {
        const base64Url = token.split('.')[1];
        const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
        const jsonPayload = decodeURIComponent(
            atob(base64)
                .split('')
                .map(c => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
                .join('')
        );
        
        return JSON.parse(jsonPayload);
    }
}
```

### **Multi-Tab Synchronization**
```typescript
export class MultiTabSyncService {
    constructor(private authService: AuthenticationService) {
        this.setupMultiTabSync();
    }
    
    private setupMultiTabSync() {
        // Listen for storage changes across tabs
        window.addEventListener('storage', (event) => {
            if (event.key === 'Token') {
                if (event.newValue === null) {
                    // Token was cleared in another tab
                    this.authService.logout();
                } else if (event.oldValue !== event.newValue) {
                    // Token was updated in another tab
                    this.handleTokenUpdate(event.newValue);
                }
            }
        });
        
        // Listen for authentication events
        this.env.getEvents().subscribe(event => {
            switch (event.Code) {
                case EVENT_TYPE.USER.LOGGED_OUT_REMOTE:
                    this.handleRemoteLogout();
                    break;
                case EVENT_TYPE.USER.STATE_CHANGED_REMOTE:
                    this.handleRemoteStateChange(event.data);
                    break;
            }
        });
    }
    
    private handleTokenUpdate(newToken: string) {
        // Update local state to match other tabs
        this.env.showMessage('Session updated from another tab', 'info');
    }
    
    private async handleRemoteLogout() {
        await this.env.showAlert(
            'You have been logged out from another session.',
            'Remote Logout',
            'Security Notice'
        );
        
        await this.authService.logout();
    }
    
    private handleRemoteStateChange(data: any) {
        console.log('Remote state change:', data);
        // Handle other state changes as needed
    }
}
```

---

## 🔒 **Security Features**

### **Device Registration**
```typescript
export class DeviceSecurityService {
    constructor(private authService: AuthenticationService) {}
    
    async registerDevice() {
        try {
            const deviceInfo = await this.getDeviceInfo();
            
            if (deviceInfo) {
                // Register device with server
                await this.commonService.connect(
                    'POST', 
                    'api/devices/register', 
                    deviceInfo
                ).toPromise();
                
                console.log('Device registered successfully');
            }
        } catch (error) {
            console.error('Device registration failed:', error);
        }
    }
    
    private async getDeviceInfo(): Promise<DeviceInfo | null> {
        try {
            if (Capacitor.isPluginAvailable('Device')) {
                const info = await Device.getInfo();
                const uid = await Device.getId();
                
                return {
                    Code: uid.identifier,
                    Name: info.name,
                    Model: info.model,
                    Platform: info.platform,
                    OperatingSystem: info.operatingSystem,
                    OsVersion: info.osVersion,
                    Manufacturer: info.manufacturer,
                    IsVirtual: info.isVirtual,
                    WebViewVersion: info.webViewVersion
                };
            }
            return null;
        } catch (error) {
            console.error('Error getting device info:', error);
            return null;
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Đúng - Comprehensive error handling
async login(credentials: LoginCredentials) {
    try {
        const result = await this.authService.login(credentials);
        return result;
    } catch (error) {
        // Handle specific error types
        if (error.status === 401) {
            this.env.showMessage('Invalid username or password', 'danger');
        } else if (error.status === 429) {
            this.env.showMessage('Too many login attempts. Please try again later.', 'warning');
        } else if (error.status === 403) {
            this.env.showMessage('Account is locked. Please contact administrator.', 'danger');
        } else {
            this.env.showErrorMessage(error);
        }
        throw error;
    }
}
```

### **2. State Management**
```typescript
// ✅ Đúng - Reactive state management
export class AuthGuardService {
    constructor(private authService: AuthenticationService) {}
    
    canActivate(): Observable<boolean> {
        return this.authService.getAuthState().pipe(
            map(state => state.isAuthenticated),
            tap(isAuthenticated => {
                if (!isAuthenticated) {
                    this.router.navigate(['/login']);
                }
            })
        );
    }
}
```

### **3. Memory Management**
```typescript
// ✅ Đúng - Proper cleanup
export class AuthComponent implements OnDestroy {
    private authSubscription: Subscription;
    
    ngOnInit() {
        this.authSubscription = this.authService.getAuthState().subscribe(state => {
            // Handle state changes
        });
    }
    
    ngOnDestroy() {
        this.authSubscription?.unsubscribe();
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Login Flow**
```typescript
// Complete login with error handling and navigation
async performLogin() {
    const result = await this.authService.login(credentials);
    if (result.success) {
        this.nav('/dashboard', 'forward');
    }
}
```

### **2. Route Protection**
```typescript
// Protect routes with authentication
@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private authService: AuthenticationService) {}
    
    canActivate(): boolean {
        return this.authService.isAuthenticated();
    }
}
```

### **3. API Request Authentication**
```typescript
// Automatic token attachment for API requests
const headers = this.authService.getAuthHeaders();
```

---

## ⚠️ **Security Considerations**

1. **Token Storage**: Tokens are stored securely in cache management
2. **Session Timeout**: Automatic logout after inactivity
3. **Multi-Tab Sync**: Consistent state across browser tabs
4. **Device Registration**: Track and manage user devices
5. **MFA Support**: Multi-factor authentication capability

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
