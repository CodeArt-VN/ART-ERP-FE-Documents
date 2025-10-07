# UserProfileService Documentation

## 📄 **Tổng quan**

`UserProfileService` là enterprise service quản lý user profile, settings, permissions, và roles trong ứng dụng ART-ERP-FE. Service này handle user data CRUD operations, profile management, và integration với authentication system.

## 🎯 **Mục đích sử dụng**

- **User Profile Management**: CRUD operations cho user profiles
- **Settings Management**: User preferences và application settings
- **Permission Integration**: Sync với permission system
- **Role Management**: Handle user roles và access rights
- **Profile Caching**: Efficient profile data caching
- **Context Setup**: Initialize user context sau khi login

## 📍 **File location**
```
src/app/services/auth/user-profile.service.ts
```

## 🔧 **Dependencies**

```typescript
import { CommonService } from '../core/common.service';
import { EnvService } from '../core/env.service';
import { UserContextService } from './user-context.service';
import { CacheManagementService } from '../core/cache-management.service';
import { SYS_StatusProvider, SYS_TypeProvider, SYS_UserSettingProvider } from '../static/services.service';
```

## 🏗️ **Interfaces**

### **UserProfile Interface**
```typescript
interface UserProfile {
    Id: number;
    Code: string;
    Name: string;
    FullName: string;
    Email: string;
    Phone: string;
    IDBranch: string;
    IDJobTitle: number;
    IsActive: boolean;
    Roles: string[];
    Permissions: string[];
    Settings: UserSettings;
}
```

### **UserSettings Interface**
```typescript
interface UserSettings {
    Theme: string;
    Language: string;
    Notifications: boolean;
    AutoSave: boolean;
    DefaultBranch: string;
    Preferences: any;
}
```

## 🔨 **Core Methods**

### **1. Profile Management**

#### **getProfile()**
Lấy user profile từ server và setup user context.

```typescript
getProfile(): Promise<void>
```

**Features:**
- Fetch fresh data từ API với menu permissions
- Process data qua UserContextService
- Update app status và type lists
- Cache profile data

**Example:**
```typescript
export class AuthService {
    constructor(private userProfileService: UserProfileService) {}

    async afterLogin() {
        try {
            await this.userProfileService.getProfile();
            console.log('User profile loaded and context setup complete');
        } catch (error) {
            console.error('Failed to load user profile:', error);
        }
    }
}
```

#### **updateProfile(profileData)**
Cập nhật user profile.

```typescript
updateProfile(profileData: Partial<UserProfile>): Promise<UserProfile>
```

**Example:**
```typescript
async updateUserInfo() {
    try {
        const updatedProfile = await this.userProfileService.updateProfile({
            FullName: 'New Full Name',
            Email: 'newemail@example.com',
            Phone: '+84901234567'
        });
        
        console.log('Profile updated:', updatedProfile);
        return updatedProfile;
    } catch (error) {
        console.error('Failed to update profile:', error);
        throw error;
    }
}
```

#### **changePassword(oldPassword, newPassword)**
Thay đổi mật khẩu user.

```typescript
changePassword(oldPassword: string, newPassword: string): Promise<void>
```

**Example:**
```typescript
async changeUserPassword(oldPass: string, newPass: string) {
    try {
        await this.userProfileService.changePassword(oldPass, newPass);
        this.env.showMessage('Password changed successfully', 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

### **2. Settings Management**

#### **getUserSettings()**
Lấy user settings từ cache hoặc server.

```typescript
getUserSettings(): Promise<UserSettings>
```

#### **updateUserSettings(settings)**
Cập nhật user settings.

```typescript
updateUserSettings(settings: Partial<UserSettings>): Promise<UserSettings>
```

#### **resetUserSettings()**
Reset user settings về default values.

```typescript
resetUserSettings(): Promise<UserSettings>
```

**Example:**
```typescript
export class UserSettingsPage {
    constructor(private userProfileService: UserProfileService) {}

    async loadSettings() {
        try {
            const settings = await this.userProfileService.getUserSettings();
            this.settingsForm.patchValue(settings);
        } catch (error) {
            console.error('Failed to load settings:', error);
        }
    }

    async saveSettings() {
        try {
            const formValue = this.settingsForm.value;
            const updatedSettings = await this.userProfileService.updateUserSettings(formValue);
            
            this.env.showMessage('Settings saved successfully', 'success');
            return updatedSettings;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async resetToDefaults() {
        try {
            const defaultSettings = await this.userProfileService.resetUserSettings();
            this.settingsForm.patchValue(defaultSettings);
            this.env.showMessage('Settings reset to defaults', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **3. Permission và Role Management**

#### **getUserPermissions()**
Lấy danh sách permissions của user hiện tại.

```typescript
getUserPermissions(): Promise<string[]>
```

#### **hasPermission(permission)**
Kiểm tra user có permission cụ thể không.

```typescript
hasPermission(permission: string): Promise<boolean>
```

#### **getUserRoles()**
Lấy danh sách roles của user.

```typescript
getUserRoles(): Promise<string[]>
```

#### **hasRole(role)**
Kiểm tra user có role cụ thể không.

```typescript
hasRole(role: string): Promise<boolean>
```

**Example:**
```typescript
export class PermissionGuard {
    constructor(private userProfileService: UserProfileService) {}

    async canAccess(requiredPermission: string): Promise<boolean> {
        try {
            return await this.userProfileService.hasPermission(requiredPermission);
        } catch (error) {
            console.error('Permission check failed:', error);
            return false;
        }
    }

    async canAccessAdminFeatures(): Promise<boolean> {
        try {
            const hasAdminRole = await this.userProfileService.hasRole('Administrator');
            const hasAdminPermission = await this.userProfileService.hasPermission('admin.access');
            
            return hasAdminRole || hasAdminPermission;
        } catch (error) {
            return false;
        }
    }
}
```

### **4. Profile Avatar và Media**

#### **uploadAvatar(file)**
Upload avatar cho user profile.

```typescript
uploadAvatar(file: File): Promise<string>
```

#### **getAvatarUrl(userId?)**
Lấy URL avatar của user.

```typescript
getAvatarUrl(userId?: number): string
```

#### **removeAvatar()**
Xóa avatar của user.

```typescript
removeAvatar(): Promise<void>
```

**Example:**
```typescript
export class ProfileAvatarComponent {
    constructor(private userProfileService: UserProfileService) {}

    async onAvatarSelected(event: any) {
        const file = event.target.files[0];
        if (!file) return;

        try {
            const avatarUrl = await this.userProfileService.uploadAvatar(file);
            this.currentAvatarUrl = avatarUrl;
            this.env.showMessage('Avatar updated successfully', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async removeCurrentAvatar() {
        try {
            await this.userProfileService.removeAvatar();
            this.currentAvatarUrl = this.getDefaultAvatarUrl();
            this.env.showMessage('Avatar removed', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    getAvatarUrl(userId?: number): string {
        return this.userProfileService.getAvatarUrl(userId);
    }
}
```

## 🎨 **Usage Examples**

### **1. Complete Profile Management**
```typescript
export class UserProfilePage extends PageBase {
    userProfile: UserProfile;
    userSettings: UserSettings;

    constructor(
        private userProfileService: UserProfileService,
        public env: EnvService
    ) {
        super();
    }

    async ionViewDidEnter() {
        await this.loadUserData();
    }

    async loadUserData() {
        try {
            // Load profile và settings
            await this.userProfileService.getProfile();
            
            this.userProfile = this.env.user;
            this.userSettings = await this.userProfileService.getUserSettings();
            
            console.log('User data loaded:', {
                profile: this.userProfile,
                settings: this.userSettings
            });
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async updateProfile() {
        try {
            const updatedProfile = await this.userProfileService.updateProfile({
                FullName: this.userProfile.FullName,
                Email: this.userProfile.Email,
                Phone: this.userProfile.Phone
            });

            this.env.showMessage('Profile updated successfully', 'success');
            this.userProfile = updatedProfile;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async updateSettings() {
        try {
            const updatedSettings = await this.userProfileService.updateUserSettings(this.userSettings);
            this.env.showMessage('Settings updated successfully', 'success');
            this.userSettings = updatedSettings;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **2. Permission-Based UI**
```typescript
export class DashboardComponent implements OnInit {
    canViewReports = false;
    canManageUsers = false;
    canAccessAdmin = false;

    constructor(private userProfileService: UserProfileService) {}

    async ngOnInit() {
        await this.checkPermissions();
    }

    async checkPermissions() {
        try {
            // Check specific permissions
            this.canViewReports = await this.userProfileService.hasPermission('reports.view');
            this.canManageUsers = await this.userProfileService.hasPermission('users.manage');
            
            // Check admin role
            this.canAccessAdmin = await this.userProfileService.hasRole('Administrator');
            
            console.log('User permissions:', {
                canViewReports: this.canViewReports,
                canManageUsers: this.canManageUsers,
                canAccessAdmin: this.canAccessAdmin
            });
        } catch (error) {
            console.error('Failed to check permissions:', error);
        }
    }
}
```

### **3. Settings Synchronization**
```typescript
export class SettingsService {
    constructor(
        private userProfileService: UserProfileService,
        private env: EnvService
    ) {}

    async syncUserSettings() {
        try {
            const settings = await this.userProfileService.getUserSettings();
            
            // Apply theme
            if (settings.Theme) {
                await this.applyTheme(settings.Theme);
            }
            
            // Apply language
            if (settings.Language) {
                await this.env.setLang(settings.Language);
            }
            
            // Apply other preferences
            if (settings.DefaultBranch) {
                this.env.changeBranch(settings.DefaultBranch);
            }
            
            console.log('User settings synchronized');
        } catch (error) {
            console.error('Failed to sync settings:', error);
        }
    }

    async updateAndSyncSetting(key: string, value: any) {
        try {
            const currentSettings = await this.userProfileService.getUserSettings();
            currentSettings[key] = value;
            
            await this.userProfileService.updateUserSettings(currentSettings);
            await this.syncUserSettings();
            
        } catch (error) {
            console.error('Failed to update setting:', error);
        }
    }

    private async applyTheme(theme: string) {
        document.body.className = `theme-${theme}`;
    }
}
```

### **4. Profile Caching Strategy**
```typescript
export class ProfileCacheService {
    constructor(
        private userProfileService: UserProfileService,
        private cacheService: CacheManagementService
    ) {}

    async getCachedProfile(forceRefresh = false): Promise<UserProfile> {
        const cacheKey = 'user_profile';
        
        if (forceRefresh) {
            await this.cacheService.remove(cacheKey);
        }
        
        try {
            let profile = await this.cacheService.get(cacheKey, 'cache-first');
            
            if (!profile) {
                await this.userProfileService.getProfile();
                profile = this.env.user;
                
                // Cache for 1 hour
                await this.cacheService.set(cacheKey, profile, {
                    enable: true,
                    timeToLive: 3600
                });
            }
            
            return profile;
        } catch (error) {
            console.error('Failed to get cached profile:', error);
            throw error;
        }
    }

    async invalidateProfileCache() {
        await this.cacheService.remove('user_profile');
        await this.cacheService.remove('user_settings');
        await this.cacheService.remove('user_permissions');
    }
}
```

### **5. Multi-User Profile Management**
```typescript
export class UserManagementService {
    constructor(private userProfileService: UserProfileService) {}

    async getUserProfiles(userIds: number[]): Promise<UserProfile[]> {
        const profiles = [];
        
        for (const userId of userIds) {
            try {
                const profile = await this.getUserProfileById(userId);
                profiles.push(profile);
            } catch (error) {
                console.error(`Failed to load profile for user ${userId}:`, error);
            }
        }
        
        return profiles;
    }

    async getUserProfileById(userId: number): Promise<UserProfile> {
        // Implementation depends on API structure
        const response = await this.commonService.connect('GET', `api/users/${userId}`).toPromise();
        return response;
    }

    async bulkUpdateProfiles(updates: Array<{userId: number, data: Partial<UserProfile>}>) {
        const results = [];
        
        for (const update of updates) {
            try {
                const result = await this.updateUserProfile(update.userId, update.data);
                results.push({ userId: update.userId, success: true, data: result });
            } catch (error) {
                results.push({ userId: update.userId, success: false, error: error.message });
            }
        }
        
        return results;
    }

    private async updateUserProfile(userId: number, data: Partial<UserProfile>): Promise<UserProfile> {
        // Implementation for updating other user profiles
        const response = await this.commonService.connect('PUT', `api/users/${userId}`, data).toPromise();
        return response;
    }
}
```

## 📋 **Best Practices**

### **1. Cache Profile Data Efficiently**
```typescript
// ✅ Cache profile với appropriate TTL
await this.cacheService.set('user_profile', profile, {
    timeToLive: 3600, // 1 hour
    autoRefresh: true
});

// ❌ Không cache hoặc cache quá lâu
// await this.cacheService.set('user_profile', profile, { timeToLive: 86400 }); // Too long
```

### **2. Handle Permission Checks Gracefully**
```typescript
// ✅ Graceful permission handling
async checkPermission(permission: string): Promise<boolean> {
    try {
        return await this.userProfileService.hasPermission(permission);
    } catch (error) {
        console.error('Permission check failed:', error);
        return false; // Fail safely
    }
}

// ❌ Không handle errors
// const hasPermission = await this.userProfileService.hasPermission(permission); // May throw
```

### **3. Sync Settings Properly**
```typescript
// ✅ Sync settings after updates
async updateTheme(theme: string) {
    await this.userProfileService.updateUserSettings({ Theme: theme });
    await this.applyTheme(theme); // Apply immediately
}

// ❌ Update without applying
// await this.userProfileService.updateUserSettings({ Theme: theme }); // Not applied
```

### **4. Invalidate Cache After Updates**
```typescript
// ✅ Invalidate cache after profile updates
async updateProfile(data: any) {
    const result = await this.userProfileService.updateProfile(data);
    await this.cacheService.remove('user_profile'); // Invalidate cache
    return result;
}
```

## 🚨 **Error Handling**

### **Common Error Scenarios**
```typescript
try {
    await this.userProfileService.getProfile();
} catch (error) {
    if (error.status === 401) {
        // Token expired
        this.router.navigate(['/login']);
    } else if (error.status === 403) {
        // Access denied
        this.env.showMessage('Access denied', 'danger');
    } else if (error.status === 404) {
        // Profile not found
        this.env.showMessage('User profile not found', 'warning');
    } else {
        // Other errors
        this.env.showErrorMessage(error);
    }
}
```

## 🔧 **Integration Points**

### **Với AuthenticationService**
```typescript
// After successful login
await this.authService.login(credentials);
await this.userProfileService.getProfile(); // Load profile
```

### **Với EnvService**
```typescript
// Profile data available in env.user
const currentUser = this.env.user;
const selectedBranch = this.env.selectedBranch;
```

### **Với CacheManagementService**
```typescript
// Profile cached in app state
const userProfile = this.cacheService.app.userProfile;
```

UserProfileService cung cấp comprehensive user management với profile CRUD, settings, permissions, và efficient caching cho ứng dụng enterprise ART-ERP-FE.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
