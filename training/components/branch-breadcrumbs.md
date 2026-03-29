# BranchBreadcrumbs Component Documentation

## 📋 **Tổng quan**

`BranchBreadcrumbsComponent` là component hiển thị breadcrumb navigation cho cấu trúc phân cấp branch/department. Component này tự động build breadcrumb path từ ID hiện tại lên đến root level và hỗ trợ collapse khi có quá nhiều levels.

**Selector**: `app-branch-breadcrumbs`  
**Location**: `src/app/components/branch-breadcrumbs/branch-breadcrumbs.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() Id: number;                     // Current branch/item ID
@Input() Items: any[];                   // Complete list of items
@Input() maxItems: number;               // Maximum items to show
@Input() itemsBeforeCollapse: number = 0; // Items before collapse
@Input() itemsAfterCollapse: number = 1;  // Items after collapse
```

### **Internal Properties**
```typescript
breadcrumbs: any[] = [];                 // Built breadcrumb path
isOpen: boolean = false;                 // Popover state
collapsedBreadcrumbs: any[] = [];        // Hidden breadcrumbs
```

---

## 🚀 **Basic Usage**

### **Simple Branch Breadcrumbs**
```typescript
// Component
export class BranchNavigationPage extends PageBase {
    currentBranchId = 15;
    branchList: any[] = [];

    async ngOnInit() {
        await this.loadBranches();
    }

    async loadBranches() {
        try {
            this.branchList = await this.env.getBranch(null, true);
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    onBranchChange(branchId: number) {
        this.currentBranchId = branchId;
        this.loadDataForBranch(branchId);
    }
}
```

```html
<!-- Template -->
<app-branch-breadcrumbs
    [Id]="currentBranchId"
    [Items]="branchList"
    [maxItems]="5"
    [itemsBeforeCollapse]="1"
    [itemsAfterCollapse]="2">
</app-branch-breadcrumbs>

<!-- Content for current branch -->
<div class="branch-content">
    <h2>{{ getCurrentBranchName() }}</h2>
    <!-- Branch-specific content -->
</div>
```

### **Department Hierarchy**
```typescript
export class DepartmentHierarchyPage extends PageBase {
    currentDepartmentId = 25;
    departments: any[] = [
        { Id: 1, Name: 'Company', IDParent: null },
        { Id: 10, Name: 'Sales Division', IDParent: 1 },
        { Id: 20, Name: 'North Region', IDParent: 10 },
        { Id: 25, Name: 'Hanoi Branch', IDParent: 20 },
        { Id: 30, Name: 'District 1 Office', IDParent: 25 }
    ];

    navigateToDepartment(departmentId: number) {
        this.currentDepartmentId = departmentId;
        this.router.navigate(['/department', departmentId]);
    }
}
```

```html
<app-branch-breadcrumbs
    [Id]="currentDepartmentId"
    [Items]="departments"
    [maxItems]="4">
</app-branch-breadcrumbs>
```

---

## 🎨 **Customization**

### **Custom Breadcrumb Styling**
```scss
// Component SCSS
app-branch-breadcrumbs {
    .breadcrumb-container {
        display: flex;
        align-items: center;
        padding: 8px 16px;
        background: var(--ion-color-light);
        border-radius: 8px;
        margin-bottom: 16px;
        
        ion-breadcrumbs {
            --color: var(--ion-color-dark);
            
            ion-breadcrumb {
                --color-active: var(--ion-color-primary);
                font-size: 0.875rem;
                
                &:hover {
                    --color: var(--ion-color-primary);
                }
            }
        }
        
        .breadcrumb-actions {
            margin-left: auto;
            
            ion-button {
                --color: var(--ion-color-medium);
                height: 24px;
            }
        }
    }
}
```

### **With Navigation Actions**
```html
<div class="breadcrumb-with-actions">
    <app-branch-breadcrumbs
        [Id]="currentBranchId"
        [Items]="branchList">
    </app-branch-breadcrumbs>
    
    <div class="breadcrumb-actions">
        <ion-button 
            fill="clear" 
            size="small"
            (click)="goToParent()">
            <ion-icon name="arrow-up"></ion-icon>
            Parent
        </ion-button>
        
        <ion-button 
            fill="clear" 
            size="small"
            (click)="showChildren()">
            <ion-icon name="arrow-down"></ion-icon>
            Children
        </ion-button>
    </div>
</div>
```

---

## 📱 **Responsive Design**

### **Mobile Optimization**
```scss
// Mobile breadcrumbs
@media (max-width: 768px) {
    app-branch-breadcrumbs {
        .breadcrumb-container {
            padding: 6px 12px;
            
            ion-breadcrumbs {
                ion-breadcrumb {
                    font-size: 0.8125rem;
                    
                    // Hide middle items on mobile
                    &:not(:first-child):not(:last-child) {
                        display: none;
                    }
                }
                
                // Show ellipsis for hidden items
                &::after {
                    content: '...';
                    margin: 0 8px;
                    color: var(--ion-color-medium);
                }
            }
        }
    }
}
```

### **Collapsible Breadcrumbs**
```typescript
export class CollapsibleBreadcrumbs {
    maxVisibleItems = 3;
    
    get shouldCollapse(): boolean {
        return this.breadcrumbs.length > this.maxVisibleItems;
    }
    
    get visibleBreadcrumbs(): any[] {
        if (!this.shouldCollapse) {
            return this.breadcrumbs;
        }
        
        return [
            this.breadcrumbs[0], // First item
            ...this.breadcrumbs.slice(-2) // Last 2 items
        ];
    }
    
    get hiddenBreadcrumbs(): any[] {
        if (!this.shouldCollapse) {
            return [];
        }
        
        return this.breadcrumbs.slice(1, -2);
    }
}
```

```html
<ion-breadcrumbs>
    <!-- First breadcrumb -->
    <ion-breadcrumb 
        *ngIf="visibleBreadcrumbs[0]"
        [routerLink]="getBreadcrumbRoute(visibleBreadcrumbs[0])">
        {{ visibleBreadcrumbs[0].Name }}
    </ion-breadcrumb>
    
    <!-- Collapsed items -->
    <ion-breadcrumb 
        *ngIf="shouldCollapse"
        (click)="showCollapsedItems($event)">
        <ion-icon name="ellipsis-horizontal"></ion-icon>
    </ion-breadcrumb>
    
    <!-- Last breadcrumbs -->
    <ion-breadcrumb 
        *ngFor="let item of visibleBreadcrumbs.slice(1)"
        [routerLink]="getBreadcrumbRoute(item)">
        {{ item.Name }}
    </ion-breadcrumb>
</ion-breadcrumbs>
```

---

## 🔄 **Advanced Features**

### **Interactive Breadcrumbs**
```typescript
export class InteractiveBreadcrumbs {
    onBreadcrumbClick(item: any, event: Event) {
        event.preventDefault();
        
        // Navigate to selected level
        this.navigateToLevel(item.Id);
    }
    
    async navigateToLevel(levelId: number) {
        try {
            // Update current level
            this.currentBranchId = levelId;
            
            // Load data for new level
            await this.loadDataForLevel(levelId);
            
            // Update URL
            this.router.navigate(['/branch', levelId]);
            
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    getBreadcrumbRoute(item: any): string {
        return `/branch/${item.Id}`;
    }
}
```

### **Context Menu Integration**
```typescript
export class BreadcrumbsWithContext {
    async showContextMenu(item: any, event: Event) {
        event.preventDefault();
        
        const actionSheet = await this.actionSheetController.create({
            header: item.Name,
            buttons: [
                {
                    text: 'View Details',
                    icon: 'information-circle',
                    handler: () => this.viewDetails(item)
                },
                {
                    text: 'Edit',
                    icon: 'create',
                    handler: () => this.editItem(item)
                },
                {
                    text: 'View Children',
                    icon: 'list',
                    handler: () => this.viewChildren(item)
                },
                {
                    text: 'Cancel',
                    icon: 'close',
                    role: 'cancel'
                }
            ]
        });
        
        await actionSheet.present();
    }
}
```

---

## 🗂️ **Data Structure**

### **Branch/Department Model**
```typescript
interface BranchItem {
    Id: number;                          // Unique identifier
    Name: string;                        // Display name
    IDParent: number | null;             // Parent ID (null for root)
    Code?: string;                       // Branch code
    Type?: string;                       // Branch type
    Level?: number;                      // Hierarchy level
    Path?: string;                       // Full path string
    IsActive?: boolean;                  // Active status
}
```

### **Building Breadcrumb Path**
```typescript
export class BreadcrumbBuilder {
    buildBreadcrumbPath(currentId: number, items: BranchItem[]): BranchItem[] {
        const path: BranchItem[] = [];
        
        const addParent = (id: number) => {
            const item = items.find(d => d.Id === id);
            if (item) {
                path.unshift(item); // Add to beginning
                if (item.IDParent) {
                    addParent(item.IDParent); // Recursive call
                }
            }
        };
        
        addParent(currentId);
        return path;
    }
    
    // Alternative: Build path with levels
    buildPathWithLevels(currentId: number, items: BranchItem[]): BranchItem[] {
        const itemsMap = new Map(items.map(item => [item.Id, item]));
        const path: BranchItem[] = [];
        
        let current = itemsMap.get(currentId);
        while (current) {
            path.unshift(current);
            current = current.IDParent ? itemsMap.get(current.IDParent) : null;
        }
        
        return path;
    }
}
```

---

## 🔧 **Integration Patterns**

### **With Page Header**
```html
<ion-header>
    <ion-toolbar>
        <ion-title>{{ pageTitle }}</ion-title>
        <ion-buttons slot="end">
            <ion-button (click)="showBranchSelector()">
                <ion-icon name="business"></ion-icon>
            </ion-button>
        </ion-buttons>
    </ion-toolbar>
</ion-header>

<ion-content>
    <app-branch-breadcrumbs
        [Id]="currentBranchId"
        [Items]="branchList">
    </app-branch-breadcrumbs>
    
    <!-- Page content -->
</ion-content>
```

### **With Data Filtering**
```typescript
export class BranchFilteredData {
    currentBranchId: number;
    filteredData: any[] = [];
    
    onBranchChange(branchId: number) {
        this.currentBranchId = branchId;
        this.filterDataByBranch(branchId);
    }
    
    async filterDataByBranch(branchId: number) {
        try {
            // Get all child branches
            const childBranches = this.getAllChildBranches(branchId);
            const branchIds = [branchId, ...childBranches.map(b => b.Id)];
            
            // Filter data
            const result = await this.dataService.read({
                BranchIds: branchIds
            });
            
            this.filteredData = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    getAllChildBranches(parentId: number): BranchItem[] {
        const children: BranchItem[] = [];
        
        const findChildren = (id: number) => {
            const directChildren = this.branchList.filter(b => b.IDParent === id);
            children.push(...directChildren);
            
            directChildren.forEach(child => {
                findChildren(child.Id);
            });
        };
        
        findChildren(parentId);
        return children;
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Cache breadcrumb paths
export class OptimizedBreadcrumbs {
    private breadcrumbCache = new Map<number, BranchItem[]>();
    
    getBreadcrumbPath(id: number): BranchItem[] {
        if (this.breadcrumbCache.has(id)) {
            return this.breadcrumbCache.get(id);
        }
        
        const path = this.buildBreadcrumbPath(id, this.branchList);
        this.breadcrumbCache.set(id, path);
        return path;
    }
    
    clearCache() {
        this.breadcrumbCache.clear();
    }
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle missing items
buildBreadcrumbPath(currentId: number, items: BranchItem[]): BranchItem[] {
    if (!currentId || !items || items.length === 0) {
        return [];
    }
    
    try {
        const path = [];
        const addParent = (id: number, depth = 0) => {
            // Prevent infinite loops
            if (depth > 10) {
                console.warn('Maximum breadcrumb depth reached');
                return;
            }
            
            const item = items.find(d => d.Id === id);
            if (item) {
                path.unshift(item);
                if (item.IDParent && item.IDParent !== id) {
                    addParent(item.IDParent, depth + 1);
                }
            }
        };
        
        addParent(currentId);
        return path;
    } catch (error) {
        console.error('Breadcrumb build error:', error);
        return [];
    }
}
```

### **3. Accessibility**
```html
<!-- ✅ Đúng - Accessible breadcrumbs -->
<nav aria-label="Breadcrumb navigation">
    <ion-breadcrumbs>
        <ion-breadcrumb 
            *ngFor="let item of breadcrumbs; let last = last"
            [attr.aria-current]="last ? 'page' : null"
            [routerLink]="!last ? getBreadcrumbRoute(item) : null">
            {{ item.Name }}
        </ion-breadcrumb>
    </ion-breadcrumbs>
</nav>
```

---

## 🚨 **Common Use Cases**

### **1. File System Navigation**
```typescript
export class FileSystemBreadcrumbs {
    currentFolderId: number;
    folders: any[] = [];
    
    navigateToFolder(folderId: number) {
        this.currentFolderId = folderId;
        this.loadFolderContents(folderId);
    }
}
```

### **2. Category Hierarchy**
```typescript
export class CategoryBreadcrumbs {
    currentCategoryId: number;
    categories: any[] = [];
    
    onCategorySelect(categoryId: number) {
        this.currentCategoryId = categoryId;
        this.filterProductsByCategory(categoryId);
    }
}
```

### **3. Organizational Structure**
```typescript
export class OrgStructureBreadcrumbs {
    currentPositionId: number;
    positions: any[] = [];
    
    viewPosition(positionId: number) {
        this.currentPositionId = positionId;
        this.loadPositionDetails(positionId);
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
