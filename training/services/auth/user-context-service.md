# UserContextService Documentation

## 📋 **Tổng quan**

`UserContextService` là enterprise-level service để manage user context, tenant switching, session management, và reactive state trong ứng dụng. Service này handle toàn bộ user-related context và provide reactive streams cho user data.

**Location**: `src/app/services/auth/user-context.service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Features**

### **Reactive State Management**
- **User Profile**: Current user information với reactive updates
- **Tenant Context**: Current tenant/organization context
- **Session Management**: Active user session tracking
- **Role Management**: User roles và permissions
- **Multi-tenant Support**: Switch between different tenants

### **Core Observables**
```typescript
private currentUser$ = new BehaviorSubject<UserProfile | null>(null);
private currentTenant$ = new BehaviorSubject<Tenant | null>(null);
private currentSession$ = new BehaviorSubject<UserSession | null>(null);
private userRoles$ = new BehaviorSubject<Role[]>([]);
```

---

## 🏗️ **Interfaces**

### **UserProfile Interface**
```typescript
interface UserProfile {
    Id: number;
    Email: string;
    FirstName: string;
    LastName: string;
    Avatar: string;
    UserName: string;
    FullName: string;
    IsDisabled: boolean;
}
```

### **Tenant Interface**
```typescript
interface Tenant {
    id: string;
    name: string;
    domain: string;
    settings: TenantSettings;
}
```

### **UserSession Interface**
```typescript
interface UserSession {
    sessionId: string;
    startTime: Date;
    lastActivity: Date;
    isActive: boolean;
}
```

---

## 🚀 **Core Methods**

### **User Management**
```typescript
setCurrentUser(user: UserProfile, updateCache: boolean = true): void
getCurrentUser(): Observable<UserProfile | null>
clearUserContext(): Promise<void>
```

### **Tenant Management**
```typescript
switchTenant(tenantId: string): Promise<void>
getCurrentTenant(): Observable<Tenant | null>
getTenantList(): Observable<Tenant[]>
```

### **Session Management**
```typescript
startSession(sessionData: UserSession): void
endSession(): Promise<void>
getCurrentSession(): Observable<UserSession | null>
```

### **Role Management**
```typescript
setUserRoles(roles: Role[]): void
getUserRoles(): Observable<Role[]>
hasRole(roleName: string): boolean
hasPermission(permission: string): boolean
```

---

## 🎨 **Usage Examples**

### **1. Basic User Context Setup**
```typescript
export class LoginService {
    constructor(private userContext: UserContextService) {}
    
    async handleSuccessfulLogin(loginResponse: any) {
        // Set user profile
        const userProfile: UserProfile = {
            Id: loginResponse.user.id,
            Email: loginResponse.user.email,
            FirstName: loginResponse.user.firstName,
            LastName: loginResponse.user.lastName,
            Avatar: loginResponse.user.avatar,
            UserName: loginResponse.user.username,
            FullName: `${loginResponse.user.firstName} ${loginResponse.user.lastName}`,
            IsDisabled: false
        };
        
        this.userContext.setCurrentUser(userProfile);
        
        // Set user roles
        this.userContext.setUserRoles(loginResponse.roles);
        
        // Start session
        const session: UserSession = {
            sessionId: loginResponse.sessionId,
            startTime: new Date(),
            lastActivity: new Date(),
            isActive: true
        };
        
        this.userContext.startSession(session);
        
        // Switch to user's default tenant
        await this.userContext.switchTenant(loginResponse.defaultTenant);
    }
}
```

### **2. Reactive User Data Subscription**
```typescript
export class HeaderComponent implements OnInit, OnDestroy {
    currentUser: UserProfile | null = null;
    currentTenant: Tenant | null = null;
    userRoles: Role[] = [];
    
    private subscriptions: Subscription[] = [];
    
    constructor(private userContext: UserContextService) {}
    
    ngOnInit() {
        // Subscribe to user changes
        this.subscriptions.push(
            this.userContext.getCurrentUser().subscribe(user => {
                this.currentUser = user;
                this.updateUserDisplay();
            })
        );
        
        // Subscribe to tenant changes
        this.subscriptions.push(
            this.userContext.getCurrentTenant().subscribe(tenant => {
                this.currentTenant = tenant;
                this.updateTenantDisplay();
            })
        );
        
        // Subscribe to role changes
        this.subscriptions.push(
            this.userContext.getUserRoles().subscribe(roles => {
                this.userRoles = roles;
                this.updatePermissions();
            })
        );
    }
    
    ngOnDestroy() {
        this.subscriptions.forEach(sub => sub.unsubscribe());
    }
    
    private updateUserDisplay() {
        if (this.currentUser) {
            console.log('User updated:', this.currentUser.FullName);
        }
    }
    
    private updateTenantDisplay() {
        if (this.currentTenant) {
            console.log('Tenant switched:', this.currentTenant.name);
        }
    }
}
```

### **3. Tenant Switching**
```typescript
export class TenantSwitcherComponent {
    availableTenants: Tenant[] = [];
    currentTenant: Tenant | null = null;
    
    constructor(private userContext: UserContextService) {}
    
    ngOnInit() {
        // Get available tenants
        this.userContext.getTenantList().subscribe(tenants => {
            this.availableTenants = tenants;
        });
        
        // Track current tenant
        this.userContext.getCurrentTenant().subscribe(tenant => {
            this.currentTenant = tenant;
        });
    }
    
    async switchToTenant(tenantId: string) {
        try {
            await this.userContext.switchTenant(tenantId);
            
            // Show success message
            this.env.showMessage('Tenant switched successfully', 'success');
            
            // Reload necessary data
            await this.reloadTenantSpecificData();
            
        } catch (error) {
            console.error('Tenant switch failed:', error);
            this.env.showErrorMessage('Failed to switch tenant');
        }
    }
    
    private async reloadTenantSpecificData() {
        // Reload branch list, user permissions, etc.
        await this.env.loadBranch();
        await this.loadUserPermissions();
    }
}
```

### **4. Permission-Based UI Control**
```typescript
export class PermissionControlledComponent {
    canViewReports = false;
    canEditUsers = false;
    canManageTenant = false;
    
    constructor(private userContext: UserContextService) {}
    
    ngOnInit() {
        this.userContext.getUserRoles().subscribe(roles => {
            this.updatePermissions(roles);
        });
    }
    
    private updatePermissions(roles: Role[]) {
        this.canViewReports = this.userContext.hasPermission('reports.view');
        this.canEditUsers = this.userContext.hasPermission('users.edit');
        this.canManageTenant = this.userContext.hasRole('TenantAdmin');
    }
    
    // Template usage
    /*
    <div *ngIf="canViewReports">
        <app-reports></app-reports>
    </div>
    
    <button *ngIf="canEditUsers" (click)="editUser()">
        Edit User
    </button>
    
    <app-tenant-settings *ngIf="canManageTenant">
    </app-tenant-settings>
    */
}
```

### **5. Session Management**
```typescript
export class SessionManagerService {
    constructor(
        private userContext: UserContextService,
        private env: EnvService
    ) {}
    
    async initializeSession(loginData: any) {
        const session: UserSession = {
            sessionId: this.generateSessionId(),
            startTime: new Date(),
            lastActivity: new Date(),
            isActive: true
        };
        
        this.userContext.startSession(session);
        
        // Setup session monitoring
        this.setupSessionMonitoring();
    }
    
    private setupSessionMonitoring() {
        // Monitor user activity
        document.addEventListener('click', () => this.updateLastActivity());
        document.addEventListener('keypress', () => this.updateLastActivity());
        
        // Check session timeout
        setInterval(() => this.checkSessionTimeout(), 60000); // Check every minute
    }
    
    private updateLastActivity() {
        this.userContext.getCurrentSession().pipe(take(1)).subscribe(session => {
            if (session) {
                session.lastActivity = new Date();
                this.userContext.startSession(session);
            }
        });
    }
    
    private async checkSessionTimeout() {
        const session = await this.userContext.getCurrentSession().pipe(take(1)).toPromise();
        
        if (session) {
            const now = new Date();
            const timeSinceActivity = now.getTime() - session.lastActivity.getTime();
            const timeoutDuration = 30 * 60 * 1000; // 30 minutes
            
            if (timeSinceActivity > timeoutDuration) {
                await this.handleSessionTimeout();
            }
        }
    }
    
    private async handleSessionTimeout() {
        this.env.showAlert('Your session has expired. Please log in again.');
        await this.userContext.endSession();
        // Redirect to login
        this.router.navigate(['/login']);
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Multi-Tenant Data Isolation**
```typescript
export class TenantDataService {
    constructor(private userContext: UserContextService) {}
    
    async getData(endpoint: string): Promise<any> {
        const tenant = await this.userContext.getCurrentTenant().pipe(take(1)).toPromise();
        
        if (!tenant) {
            throw new Error('No tenant context available');
        }
        
        // Add tenant context to API calls
        const tenantSpecificEndpoint = `${endpoint}?tenantId=${tenant.id}`;
        return this.http.get(tenantSpecificEndpoint);
    }
    
    async saveData(data: any): Promise<any> {
        const tenant = await this.userContext.getCurrentTenant().pipe(take(1)).toPromise();
        
        // Ensure data includes tenant context
        const tenantData = {
            ...data,
            tenantId: tenant?.id
        };
        
        return this.http.post('/api/data', tenantData);
    }
}
```

### **2. Role-Based Route Guards**
```typescript
@Injectable()
export class RoleGuard implements CanActivate {
    constructor(private userContext: UserContextService) {}
    
    canActivate(route: ActivatedRouteSnapshot): Observable<boolean> {
        const requiredRoles = route.data['roles'] as string[];
        
        return this.userContext.getUserRoles().pipe(
            map(userRoles => {
                if (!requiredRoles || requiredRoles.length === 0) {
                    return true;
                }
                
                return requiredRoles.some(role => 
                    this.userContext.hasRole(role)
                );
            })
        );
    }
}

// Route configuration
const routes: Routes = [
    {
        path: 'admin',
        component: AdminComponent,
        canActivate: [RoleGuard],
        data: { roles: ['Admin', 'SuperAdmin'] }
    }
];
```

### **3. User Context Persistence**
```typescript
export class UserContextPersistenceService {
    constructor(
        private userContext: UserContextService,
        private storage: StorageService
    ) {}
    
    async saveUserContext() {
        const user = await this.userContext.getCurrentUser().pipe(take(1)).toPromise();
        const tenant = await this.userContext.getCurrentTenant().pipe(take(1)).toPromise();
        const roles = await this.userContext.getUserRoles().pipe(take(1)).toPromise();
        
        const contextData = {
            user,
            tenant,
            roles,
            timestamp: new Date().toISOString()
        };
        
        await this.storage.set('userContext', contextData);
    }
    
    async restoreUserContext(): Promise<boolean> {
        try {
            const contextData = await this.storage.get('userContext');
            
            if (contextData && this.isContextValid(contextData)) {
                this.userContext.setCurrentUser(contextData.user, false);
                await this.userContext.switchTenant(contextData.tenant.id);
                this.userContext.setUserRoles(contextData.roles);
                
                return true;
            }
        } catch (error) {
            console.error('Failed to restore user context:', error);
        }
        
        return false;
    }
    
    private isContextValid(contextData: any): boolean {
        const savedTime = new Date(contextData.timestamp);
        const now = new Date();
        const hoursSinceLastSave = (now.getTime() - savedTime.getTime()) / (1000 * 60 * 60);
        
        return hoursSinceLastSave < 24; // Context valid for 24 hours
    }
}
```

---

## 📋 **Best Practices**

### **1. Reactive Programming**
```typescript
// ✅ Use reactive patterns
export class UserDependentComponent {
    user$ = this.userContext.getCurrentUser();
    tenant$ = this.userContext.getCurrentTenant();
    
    // Combine multiple streams
    userTenantInfo$ = combineLatest([
        this.user$,
        this.tenant$
    ]).pipe(
        map(([user, tenant]) => ({ user, tenant }))
    );
    
    constructor(private userContext: UserContextService) {}
}

// ❌ Avoid direct property access
export class BadComponent {
    ngOnInit() {
        // Don't do this - not reactive
        const user = this.userContext.currentUser;
    }
}
```

### **2. Memory Management**
```typescript
// ✅ Proper subscription management
export class ComponentWithSubscriptions implements OnDestroy {
    private destroy$ = new Subject<void>();
    
    ngOnInit() {
        this.userContext.getCurrentUser()
            .pipe(takeUntil(this.destroy$))
            .subscribe(user => {
                // Handle user changes
            });
    }
    
    ngOnDestroy() {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
```

### **3. Error Handling**
```typescript
// ✅ Handle context errors gracefully
async switchTenantSafely(tenantId: string) {
    try {
        await this.userContext.switchTenant(tenantId);
    } catch (error) {
        console.error('Tenant switch failed:', error);
        
        // Fallback to default tenant
        await this.userContext.switchTenant('default');
        
        // Notify user
        this.env.showErrorMessage('Failed to switch tenant. Using default.');
    }
}
```

---

## 🚨 **Security Considerations**

### **1. Data Validation**
```typescript
setCurrentUser(user: UserProfile, updateCache: boolean = true): void {
    // Validate user data
    if (!this.isValidUserProfile(user)) {
        throw new Error('Invalid user profile data');
    }
    
    // Sanitize user data
    const sanitizedUser = this.sanitizeUserProfile(user);
    
    this.currentUser$.next(sanitizedUser);
    
    if (updateCache) {
        this.cache.app.userProfile = sanitizedUser;
    }
}

private isValidUserProfile(user: UserProfile): boolean {
    return user && 
           typeof user.Id === 'number' && 
           typeof user.Email === 'string' &&
           user.Email.includes('@');
}
```

### **2. Permission Validation**
```typescript
hasPermission(permission: string): boolean {
    const roles = this.userRoles$.value;
    
    // Validate permission format
    if (!permission || typeof permission !== 'string') {
        return false;
    }
    
    // Check if user has required permission
    return roles.some(role => 
        role.permissions && 
        role.permissions.includes(permission)
    );
}
```

---

## 🎯 **Integration Examples**

### **With Authentication Service**
```typescript
export class AuthIntegrationService {
    constructor(
        private auth: AuthenticationService,
        private userContext: UserContextService
    ) {
        // Listen to auth state changes
        this.auth.getAuthState().subscribe(authState => {
            if (authState.isAuthenticated && authState.user) {
                this.userContext.setCurrentUser(authState.user);
            } else {
                this.userContext.clearUserContext();
            }
        });
    }
}
```

### **With Environment Service**
```typescript
export class EnvIntegrationService {
    constructor(
        private userContext: UserContextService,
        private env: EnvService
    ) {
        // Update env service when tenant changes
        this.userContext.getCurrentTenant().subscribe(tenant => {
            if (tenant) {
                this.env.selectedBranch = tenant.defaultBranch;
            }
        });
    }
}
```

UserContextService cung cấp comprehensive solution cho enterprise user management với reactive state, multi-tenant support, và robust session management trong Angular applications.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
