# ExternalAuthService Documentation

## 📋 **Tổng quan**

`ExternalAuthService` là enterprise-level service để handle OAuth2, social logins, và external identity providers. Service này support multiple OAuth providers như Google, Microsoft, Facebook và provide unified interface cho external authentication.

**Location**: `src/app/services/auth/external-auth.service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Features**

### **OAuth2 Provider Support**
- **Google OAuth**: Google account integration
- **Microsoft OAuth**: Microsoft/Azure AD integration  
- **Facebook OAuth**: Facebook account integration
- **Custom Providers**: Extensible for additional providers
- **PKCE Support**: Proof Key for Code Exchange for security

### **OAuth Provider Configuration**
```typescript
interface OAuthProvider {
    name: string;
    clientId: string;
    redirectUri: string;
    scope: string[];
    authUrl: string;
    tokenUrl: string;
}
```

---

## 🏗️ **Supported Providers**

### **Google OAuth**
```typescript
google: {
    name: 'Google',
    clientId: '', // Configured from environment
    redirectUri: `${window.location.origin}/auth/callback/google`,
    scope: ['openid', 'email', 'profile'],
    authUrl: 'https://accounts.google.com/oauth/authorize',
    tokenUrl: 'https://oauth2.googleapis.com/token'
}
```

### **Microsoft OAuth**
```typescript
microsoft: {
    name: 'Microsoft',
    clientId: '', // Configured from environment
    redirectUri: `${window.location.origin}/auth/callback/microsoft`,
    scope: ['openid', 'email', 'profile'],
    authUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
    tokenUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
}
```

### **Facebook OAuth**
```typescript
facebook: {
    name: 'Facebook',
    clientId: '', // Configured from environment
    redirectUri: `${window.location.origin}/auth/callback/facebook`,
    scope: ['email', 'public_profile'],
    authUrl: 'https://www.facebook.com/v18.0/dialog/oauth',
    tokenUrl: 'https://graph.facebook.com/v18.0/oauth/access_token'
}
```

---

## 🚀 **Core Methods**

### **OAuth Flow Methods**
```typescript
initiateOAuth(provider: string): Promise<void>
handleOAuthCallback(provider: string, code: string, state?: string): Promise<ExternalAuthResult>
exchangeCodeForToken(provider: string, code: string): Promise<TokenResponse>
getUserProfile(provider: string, accessToken: string): Promise<any>
```

### **Provider Management**
```typescript
getAvailableProviders(): OAuthProvider[]
isProviderSupported(provider: string): boolean
configureProvider(provider: string, config: Partial<OAuthProvider>): void
```

### **Security Methods**
```typescript
generateState(): string
validateState(receivedState: string, expectedState: string): boolean
generatePKCE(): { codeVerifier: string, codeChallenge: string }
```

---

## 🎨 **Usage Examples**

### **1. Basic OAuth Login**
```typescript
export class LoginComponent {
    constructor(private externalAuth: ExternalAuthService) {}
    
    async loginWithGoogle() {
        try {
            // Initiate OAuth flow
            await this.externalAuth.initiateOAuth('google');
            
            // User will be redirected to Google
            // Callback will be handled by callback component
            
        } catch (error) {
            console.error('Google login failed:', error);
            this.env.showErrorMessage('Google login failed');
        }
    }
    
    async loginWithMicrosoft() {
        try {
            await this.externalAuth.initiateOAuth('microsoft');
        } catch (error) {
            console.error('Microsoft login failed:', error);
            this.env.showErrorMessage('Microsoft login failed');
        }
    }
    
    async loginWithFacebook() {
        try {
            await this.externalAuth.initiateOAuth('facebook');
        } catch (error) {
            console.error('Facebook login failed:', error);
            this.env.showErrorMessage('Facebook login failed');
        }
    }
}
```

### **2. OAuth Callback Handling**
```typescript
export class AuthCallbackComponent implements OnInit {
    constructor(
        private route: ActivatedRoute,
        private externalAuth: ExternalAuthService,
        private auth: AuthenticationService,
        private router: Router
    ) {}
    
    async ngOnInit() {
        const provider = this.route.snapshot.params['provider'];
        const code = this.route.snapshot.queryParams['code'];
        const state = this.route.snapshot.queryParams['state'];
        const error = this.route.snapshot.queryParams['error'];
        
        if (error) {
            this.handleOAuthError(error);
            return;
        }
        
        if (code && provider) {
            await this.handleOAuthSuccess(provider, code, state);
        } else {
            this.handleOAuthError('Missing authorization code');
        }
    }
    
    private async handleOAuthSuccess(provider: string, code: string, state?: string) {
        try {
            // Handle OAuth callback
            const result = await this.externalAuth.handleOAuthCallback(provider, code, state);
            
            if (result.success) {
                // Complete authentication with your backend
                const authResult = await this.auth.loginWithExternalProvider({
                    provider: provider,
                    accessToken: result.accessToken,
                    userProfile: result.userProfile
                });
                
                if (authResult.success) {
                    this.env.showMessage('Login successful', 'success');
                    this.router.navigate(['/dashboard']);
                } else {
                    throw new Error('Backend authentication failed');
                }
            }
            
        } catch (error) {
            console.error('OAuth callback failed:', error);
            this.handleOAuthError(error.message);
        }
    }
    
    private handleOAuthError(error: string) {
        console.error('OAuth error:', error);
        this.env.showErrorMessage(`Login failed: ${error}`);
        this.router.navigate(['/login']);
    }
}
```

### **3. Provider Configuration**
```typescript
export class ExternalAuthConfigService {
    constructor(private externalAuth: ExternalAuthService) {}
    
    configureProviders() {
        // Configure Google OAuth
        this.externalAuth.configureProvider('google', {
            clientId: environment.oauth.google.clientId,
            scope: ['openid', 'email', 'profile', 'https://www.googleapis.com/auth/calendar.readonly']
        });
        
        // Configure Microsoft OAuth
        this.externalAuth.configureProvider('microsoft', {
            clientId: environment.oauth.microsoft.clientId,
            scope: ['openid', 'email', 'profile', 'User.Read']
        });
        
        // Configure Facebook OAuth
        this.externalAuth.configureProvider('facebook', {
            clientId: environment.oauth.facebook.clientId,
            scope: ['email', 'public_profile', 'user_friends']
        });
    }
    
    getAvailableProviders() {
        return this.externalAuth.getAvailableProviders()
            .filter(provider => provider.clientId); // Only show configured providers
    }
}
```

### **4. Advanced OAuth with PKCE**
```typescript
export class SecureOAuthService {
    constructor(private externalAuth: ExternalAuthService) {}
    
    async initiateSecureOAuth(provider: string) {
        try {
            // Generate PKCE parameters
            const pkce = this.externalAuth.generatePKCE();
            
            // Store code verifier securely
            await this.storageService.set(`pkce_${provider}`, pkce.codeVerifier);
            
            // Generate state for CSRF protection
            const state = this.externalAuth.generateState();
            await this.storageService.set(`oauth_state_${provider}`, state);
            
            // Build authorization URL with PKCE
            const authUrl = this.buildAuthUrl(provider, {
                codeChallenge: pkce.codeChallenge,
                codeChallengeMethod: 'S256',
                state: state
            });
            
            // Redirect to OAuth provider
            window.location.href = authUrl;
            
        } catch (error) {
            console.error('Secure OAuth initiation failed:', error);
            throw error;
        }
    }
    
    async handleSecureCallback(provider: string, code: string, state: string) {
        try {
            // Validate state
            const expectedState = await this.storageService.get(`oauth_state_${provider}`);
            if (!this.externalAuth.validateState(state, expectedState)) {
                throw new Error('Invalid state parameter');
            }
            
            // Get code verifier
            const codeVerifier = await this.storageService.get(`pkce_${provider}`);
            if (!codeVerifier) {
                throw new Error('Missing code verifier');
            }
            
            // Exchange code for token with PKCE
            const tokenResponse = await this.exchangeCodeWithPKCE(provider, code, codeVerifier);
            
            // Clean up stored values
            await this.storageService.remove(`pkce_${provider}`);
            await this.storageService.remove(`oauth_state_${provider}`);
            
            return tokenResponse;
            
        } catch (error) {
            console.error('Secure OAuth callback failed:', error);
            throw error;
        }
    }
}
```

### **5. Multi-Provider Login UI**
```typescript
export class MultiProviderLoginComponent {
    availableProviders: OAuthProvider[] = [];
    
    constructor(private externalAuth: ExternalAuthService) {}
    
    ngOnInit() {
        this.availableProviders = this.externalAuth.getAvailableProviders()
            .filter(provider => provider.clientId);
    }
    
    async loginWithProvider(providerName: string) {
        try {
            await this.externalAuth.initiateOAuth(providerName);
        } catch (error) {
            console.error(`${providerName} login failed:`, error);
            this.env.showErrorMessage(`${providerName} login failed`);
        }
    }
    
    getProviderIcon(providerName: string): string {
        const icons = {
            google: 'logo-google',
            microsoft: 'logo-microsoft',
            facebook: 'logo-facebook'
        };
        return icons[providerName] || 'log-in-outline';
    }
}
```

```html
<!-- multi-provider-login.component.html -->
<div class="provider-buttons">
    <ion-button 
        *ngFor="let provider of availableProviders"
        (click)="loginWithProvider(provider.name.toLowerCase())"
        fill="outline"
        expand="block">
        
        <ion-icon 
            [name]="getProviderIcon(provider.name.toLowerCase())" 
            slot="start">
        </ion-icon>
        
        Continue with {{ provider.name }}
    </ion-button>
</div>
```

---

## ⚙️ **Configuration**

### **Environment Configuration**
```typescript
// environment.ts
export const environment = {
    oauth: {
        google: {
            clientId: 'your-google-client-id.apps.googleusercontent.com',
            enabled: true
        },
        microsoft: {
            clientId: 'your-microsoft-client-id',
            tenantId: 'common', // or specific tenant ID
            enabled: true
        },
        facebook: {
            clientId: 'your-facebook-app-id',
            enabled: true
        }
    },
    externalAuth: {
        enablePKCE: true,
        stateValidation: true,
        tokenValidation: true
    }
};
```

### **Route Configuration**
```typescript
const routes: Routes = [
    {
        path: 'auth/callback/:provider',
        component: AuthCallbackComponent
    },
    {
        path: 'login',
        component: LoginComponent
    }
];
```

---

## 🔧 **Advanced Features**

### **1. Token Refresh**
```typescript
export class TokenRefreshService {
    constructor(private externalAuth: ExternalAuthService) {}
    
    async refreshExternalToken(provider: string, refreshToken: string): Promise<TokenResponse> {
        try {
            const response = await this.http.post(`/api/auth/refresh/${provider}`, {
                refreshToken: refreshToken
            }).toPromise();
            
            return response;
        } catch (error) {
            console.error('Token refresh failed:', error);
            throw error;
        }
    }
    
    async isTokenExpired(token: string): Promise<boolean> {
        try {
            const payload = JSON.parse(atob(token.split('.')[1]));
            const now = Math.floor(Date.now() / 1000);
            return payload.exp < now;
        } catch (error) {
            return true; // Assume expired if can't parse
        }
    }
}
```

### **2. Provider-Specific API Calls**
```typescript
export class ProviderApiService {
    constructor(private externalAuth: ExternalAuthService) {}
    
    async getGoogleCalendarEvents(accessToken: string): Promise<any> {
        const headers = new HttpHeaders({
            'Authorization': `Bearer ${accessToken}`
        });
        
        return this.http.get('https://www.googleapis.com/calendar/v3/calendars/primary/events', {
            headers
        }).toPromise();
    }
    
    async getMicrosoftProfile(accessToken: string): Promise<any> {
        const headers = new HttpHeaders({
            'Authorization': `Bearer ${accessToken}`
        });
        
        return this.http.get('https://graph.microsoft.com/v1.0/me', {
            headers
        }).toPromise();
    }
    
    async getFacebookFriends(accessToken: string): Promise<any> {
        return this.http.get(`https://graph.facebook.com/me/friends?access_token=${accessToken}`)
            .toPromise();
    }
}
```

### **3. Error Handling & Retry Logic**
```typescript
export class RobustExternalAuthService {
    constructor(private externalAuth: ExternalAuthService) {}
    
    async loginWithRetry(provider: string, maxRetries: number = 3): Promise<ExternalAuthResult> {
        let lastError: any;
        
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                await this.externalAuth.initiateOAuth(provider);
                return; // Success, exit retry loop
                
            } catch (error) {
                lastError = error;
                console.warn(`OAuth attempt ${attempt} failed:`, error);
                
                if (attempt < maxRetries) {
                    // Wait before retry (exponential backoff)
                    const delay = Math.pow(2, attempt) * 1000;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }
        
        throw new Error(`OAuth failed after ${maxRetries} attempts: ${lastError.message}`);
    }
    
    async handleProviderError(provider: string, error: any): Promise<void> {
        const errorHandlers = {
            'access_denied': () => {
                this.env.showMessage('Access denied by user', 'warning');
            },
            'invalid_request': () => {
                this.env.showErrorMessage('Invalid OAuth request');
            },
            'server_error': async () => {
                this.env.showErrorMessage('Provider server error. Please try again.');
                // Retry after delay
                setTimeout(() => this.loginWithRetry(provider), 5000);
            }
        };
        
        const handler = errorHandlers[error.error] || (() => {
            this.env.showErrorMessage(`OAuth error: ${error.error_description || error.message}`);
        });
        
        handler();
    }
}
```

---

## 📋 **Best Practices**

### **1. Security Best Practices**
```typescript
// ✅ Always use HTTPS for OAuth redirects
const redirectUri = `https://${window.location.host}/auth/callback/${provider}`;

// ✅ Validate state parameter
if (!this.externalAuth.validateState(receivedState, expectedState)) {
    throw new Error('Potential CSRF attack detected');
}

// ✅ Use PKCE for public clients
const pkce = this.externalAuth.generatePKCE();

// ✅ Validate tokens on backend
const validationResult = await this.validateTokenOnBackend(accessToken);
```

### **2. Error Handling**
```typescript
// ✅ Comprehensive error handling
try {
    await this.externalAuth.initiateOAuth(provider);
} catch (error) {
    if (error.code === 'POPUP_BLOCKED') {
        this.env.showAlert('Please allow popups for OAuth login');
    } else if (error.code === 'NETWORK_ERROR') {
        this.env.showErrorMessage('Network error. Please check your connection.');
    } else {
        this.env.showErrorMessage('Login failed. Please try again.');
    }
}
```

### **3. User Experience**
```typescript
// ✅ Show loading states
async loginWithProvider(provider: string) {
    const loading = await this.loadingController.create({
        message: `Connecting to ${provider}...`
    });
    
    await loading.present();
    
    try {
        await this.externalAuth.initiateOAuth(provider);
    } finally {
        await loading.dismiss();
    }
}
```

---

## 🚨 **Security Considerations**

### **1. CSRF Protection**
```typescript
// Always use state parameter
const state = this.generateSecureState();
await this.storageService.set('oauth_state', state);

// Validate state on callback
const receivedState = this.route.snapshot.queryParams['state'];
const expectedState = await this.storageService.get('oauth_state');

if (receivedState !== expectedState) {
    throw new Error('Invalid state parameter - potential CSRF attack');
}
```

### **2. Token Security**
```typescript
// Store tokens securely
await this.secureStorage.set('oauth_token', accessToken);

// Never log sensitive data
// ❌ console.log('Access token:', accessToken);
// ✅ console.log('OAuth login successful');

// Validate tokens server-side
const isValid = await this.validateTokenOnServer(accessToken);
```

### **3. Scope Validation**
```typescript
// Request minimal necessary scopes
const scopes = ['openid', 'email', 'profile']; // Minimal scopes

// Validate received scopes
if (!this.validateReceivedScopes(receivedScopes, requestedScopes)) {
    throw new Error('Insufficient permissions granted');
}
```

---

## 📱 **Mobile Considerations**

### **Capacitor Integration**
```typescript
import { Browser } from '@capacitor/browser';

async initiateOAuthMobile(provider: string) {
    if (Capacitor.isNativePlatform()) {
        // Use in-app browser for mobile
        await Browser.open({
            url: this.buildAuthUrl(provider),
            windowName: '_self'
        });
    } else {
        // Use popup for web
        window.open(this.buildAuthUrl(provider), '_blank');
    }
}
```

### **Deep Link Handling**
```typescript
// Handle OAuth callback via deep links
export class DeepLinkHandler {
    constructor(private externalAuth: ExternalAuthService) {
        App.addListener('appUrlOpen', (data) => {
            this.handleDeepLink(data.url);
        });
    }
    
    private async handleDeepLink(url: string) {
        const urlObj = new URL(url);
        
        if (urlObj.pathname.includes('/auth/callback/')) {
            const provider = urlObj.pathname.split('/').pop();
            const code = urlObj.searchParams.get('code');
            const state = urlObj.searchParams.get('state');
            
            if (code && provider) {
                await this.externalAuth.handleOAuthCallback(provider, code, state);
            }
        }
    }
}
```

ExternalAuthService cung cấp comprehensive OAuth2 solution với support cho multiple providers, security best practices, và enterprise-grade features cho external authentication trong Angular applications.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
