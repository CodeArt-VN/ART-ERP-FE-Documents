# FormManagementService Documentation

## 📋 **Tổng quan**

`FormManagementService` là service để handle form management operations, đặc biệt là tạo dynamic select data sources với search functionality. Service này provide reactive search capabilities cho các form controls như select boxes, autocomplete, và tree selectors.

**Location**: `src/app/services/page/form-management.service.ts`  
**Type**: Class (not Injectable - intended for instantiation)

---

## 🔧 **Key Features**

### **Dynamic Data Sources**
- **Reactive Search**: Tạo reactive search data sources
- **Debounced Input**: Debounce user input để optimize API calls
- **Loading States**: Manage loading states cho search operations
- **Error Handling**: Graceful error handling với empty fallbacks
- **Tree Building**: Support cho tree structure data với `buildFlatTree`

### **Search Functionality**
- **Async Search**: Support async search functions
- **Caching**: Cache selected items để improve performance
- **Distinct Values**: Prevent duplicate search requests
- **Observable Streams**: RxJS-based reactive programming

---

## 🚀 **Core Methods**

### **Create Select Data Source**
```typescript
createSelectDataSource(
    searchFunction: (term: string) => Observable<any>,
    buildFlatTree: boolean = false
): SelectDataSource
```

**Parameters**:
- `searchFunction`: Function để search data, return Observable
- `buildFlatTree`: Boolean để enable tree structure building

**Returns**: SelectDataSource object với search capabilities

---

## 🏗️ **SelectDataSource Interface**

```typescript
interface SelectDataSource {
    searchFunction: (term: string) => Observable<any>;
    loading: boolean;
    input$: Subject<string>;
    selected: any[];
    items$: Observable<any>;
    initSearch(): void;
}
```

---

## 🎨 **Usage Examples**

### **1. Basic Select with Search**
```typescript
export class CustomerFormComponent {
    customerDataSource: any;
    
    constructor(private formManager: FormManagementService) {
        this.setupCustomerSearch();
    }
    
    setupCustomerSearch() {
        // Create search function
        const searchCustomers = (term: string) => {
            return this.customerService.search({
                searchTerm: term,
                limit: 20
            }).pipe(
                map(result => result.data || [])
            );
        };
        
        // Create data source
        this.customerDataSource = this.formManager.createSelectDataSource(searchCustomers);
        
        // Initialize search
        this.customerDataSource.initSearch();
    }
    
    onCustomerSearch(term: string) {
        this.customerDataSource.input$.next(term);
    }
    
    onCustomerSelect(customer: any) {
        this.customerDataSource.selected = [customer];
        this.form.patchValue({ customerId: customer.Id });
    }
}
```

### **2. Tree Structure Select**
```typescript
export class CategoryFormComponent {
    categoryDataSource: any;
    
    constructor(private formManager: FormManagementService) {
        this.setupCategorySearch();
    }
    
    setupCategorySearch() {
        // Search function for hierarchical categories
        const searchCategories = (term: string) => {
            return this.categoryService.search({
                searchTerm: term,
                includeChildren: true
            }).pipe(
                map(result => result.data || [])
            );
        };
        
        // Create data source with tree building enabled
        this.categoryDataSource = this.formManager.createSelectDataSource(
            searchCategories,
            true // Enable buildFlatTree
        );
        
        this.categoryDataSource.initSearch();
    }
    
    onCategorySearch(term: string) {
        this.categoryDataSource.input$.next(term);
    }
}
```

### **3. Enhanced Form Management Service**
```typescript
export class EnhancedFormManagementService extends FormManagementService {
    private cache = new Map<string, any>();
    private cacheTimeout = 5 * 60 * 1000; // 5 minutes
    
    createCachedSelectDataSource(
        searchFunction: (term: string) => Observable<any>,
        cacheKey: string,
        buildFlatTree: boolean = false
    ): any {
        const cachedSearchFunction = (term: string) => {
            const fullCacheKey = `${cacheKey}_${term}`;
            const cached = this.cache.get(fullCacheKey);
            
            // Return cached data if valid
            if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
                return of(cached.data);
            }
            
            // Fetch fresh data
            return searchFunction(term).pipe(
                tap(data => {
                    // Cache the result
                    this.cache.set(fullCacheKey, {
                        data: data,
                        timestamp: Date.now()
                    });
                })
            );
        };
        
        return this.createSelectDataSource(cachedSearchFunction, buildFlatTree);
    }
    
    createMultiSelectDataSource(
        searchFunction: (term: string) => Observable<any>,
        buildFlatTree: boolean = false
    ): any {
        const dataSource = this.createSelectDataSource(searchFunction, buildFlatTree);
        
        // Extend for multi-select functionality
        return {
            ...dataSource,
            selectedItems: [],
            
            addSelection(item: any) {
                if (!this.selectedItems.find(selected => selected.Id === item.Id)) {
                    this.selectedItems.push(item);
                }
            },
            
            removeSelection(itemId: string) {
                this.selectedItems = this.selectedItems.filter(item => item.Id !== itemId);
            },
            
            clearSelections() {
                this.selectedItems = [];
            },
            
            getSelectedIds(): string[] {
                return this.selectedItems.map(item => item.Id);
            }
        };
    }
    
    createDependentSelectDataSource(
        searchFunction: (term: string, parentValue: any) => Observable<any>,
        parentControl: AbstractControl,
        buildFlatTree: boolean = false
    ): any {
        const dependentSearchFunction = (term: string) => {
            const parentValue = parentControl.value;
            
            if (!parentValue) {
                return of([]);
            }
            
            return searchFunction(term, parentValue);
        };
        
        const dataSource = this.createSelectDataSource(dependentSearchFunction, buildFlatTree);
        
        // Reset when parent changes
        parentControl.valueChanges.subscribe(() => {
            dataSource.selected = [];
            dataSource.input$.next('');
        });
        
        return dataSource;
    }
    
    createAsyncValidatedSelectDataSource(
        searchFunction: (term: string) => Observable<any>,
        validationFunction: (item: any) => Observable<boolean>,
        buildFlatTree: boolean = false
    ): any {
        const validatedSearchFunction = (term: string) => {
            return searchFunction(term).pipe(
                switchMap(items => {
                    // Validate each item
                    const validationPromises = items.map(item =>
                        validationFunction(item).pipe(
                            map(isValid => ({ ...item, isValid }))
                        )
                    );
                    
                    return forkJoin(validationPromises);
                }),
                map(validatedItems => validatedItems.filter(item => item.isValid))
            );
        };
        
        return this.createSelectDataSource(validatedSearchFunction, buildFlatTree);
    }
}
```

### **4. Advanced Search Components**
```typescript
export class AdvancedSearchComponent {
    searchDataSources: { [key: string]: any } = {};
    
    constructor(private formManager: EnhancedFormManagementService) {
        this.setupSearchDataSources();
    }
    
    setupSearchDataSources() {
        // Product search with caching
        this.searchDataSources.products = this.formManager.createCachedSelectDataSource(
            (term) => this.productService.search({ searchTerm: term }),
            'products'
        );
        
        // Customer search with validation
        this.searchDataSources.customers = this.formManager.createAsyncValidatedSelectDataSource(
            (term) => this.customerService.search({ searchTerm: term }),
            (customer) => this.customerService.validateCustomer(customer.Id)
        );
        
        // Category tree search
        this.searchDataSources.categories = this.formManager.createSelectDataSource(
            (term) => this.categoryService.search({ searchTerm: term }),
            true // Enable tree building
        );
        
        // Dependent location search (country -> state -> city)
        this.searchDataSources.states = this.formManager.createDependentSelectDataSource(
            (term, country) => this.locationService.searchStates(term, country.Id),
            this.form.get('country')
        );
        
        this.searchDataSources.cities = this.formManager.createDependentSelectDataSource(
            (term, state) => this.locationService.searchCities(term, state.Id),
            this.form.get('state')
        );
        
        // Initialize all searches
        Object.values(this.searchDataSources).forEach(dataSource => {
            dataSource.initSearch();
        });
    }
    
    onSearch(dataSourceKey: string, term: string) {
        const dataSource = this.searchDataSources[dataSourceKey];
        if (dataSource) {
            dataSource.input$.next(term);
        }
    }
    
    onSelect(dataSourceKey: string, item: any) {
        const dataSource = this.searchDataSources[dataSourceKey];
        if (dataSource) {
            dataSource.selected = [item];
            
            // Update form control
            this.form.patchValue({ [dataSourceKey]: item.Id });
            
            // Emit selection event
            this.env.publishEvent({
                Code: 'form:select',
                Value: { field: dataSourceKey, item: item }
            });
        }
    }
}
```

### **5. Form Builder Integration**
```typescript
export class DynamicFormBuilderService {
    constructor(private formManager: FormManagementService) {}
    
    buildDynamicForm(formConfig: any): FormGroup {
        const formControls = {};
        const dataSources = {};
        
        formConfig.fields.forEach(field => {
            // Create form control
            formControls[field.name] = new FormControl(
                field.defaultValue,
                field.validators || []
            );
            
            // Create data source for select fields
            if (field.type === 'select' && field.searchConfig) {
                dataSources[field.name] = this.createFieldDataSource(field.searchConfig);
            }
        });
        
        const formGroup = new FormGroup(formControls);
        
        // Attach data sources to form
        (formGroup as any).dataSources = dataSources;
        
        return formGroup;
    }
    
    private createFieldDataSource(searchConfig: any): any {
        const searchFunction = (term: string) => {
            return this.http.get(searchConfig.endpoint, {
                params: { search: term, ...searchConfig.params }
            }).pipe(
                map(response => response.data || [])
            );
        };
        
        return this.formManager.createSelectDataSource(
            searchFunction,
            searchConfig.buildFlatTree || false
        );
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Search Optimization**
```typescript
export class OptimizedFormManagementService extends FormManagementService {
    createOptimizedSelectDataSource(
        searchFunction: (term: string) => Observable<any>,
        options: {
            debounceTime?: number;
            minSearchLength?: number;
            maxResults?: number;
            buildFlatTree?: boolean;
        } = {}
    ): any {
        const {
            debounceTime = 300,
            minSearchLength = 2,
            maxResults = 50,
            buildFlatTree = false
        } = options;
        
        const optimizedSearchFunction = (term: string) => {
            // Skip search if term is too short
            if (term.length < minSearchLength) {
                return of([]);
            }
            
            return searchFunction(term).pipe(
                map(results => results.slice(0, maxResults))
            );
        };
        
        const dataSource = {
            searchFunction: optimizedSearchFunction,
            loading: false,
            input$: new Subject<string>(),
            selected: [],
            items$: null,
            
            initSearch() {
                this.loading = false;
                this.items$ = concat(
                    of(this.selected),
                    this.input$.pipe(
                        distinctUntilChanged(),
                        debounceTime(debounceTime), // Custom debounce time
                        tap(() => (this.loading = true)),
                        switchMap((term) => {
                            return this.searchFunction(term).pipe(
                                catchError(() => of([])),
                                tap(() => (this.loading = false)),
                                mergeMap((e: any) => {
                                    if (buildFlatTree) {
                                        return lib.buildFlatTree(e, e);
                                    }
                                    return Promise.resolve(e);
                                })
                            );
                        })
                    )
                );
            }
        };
        
        return dataSource;
    }
}
```

### **2. Form Validation Integration**
```typescript
export class ValidatedFormManagementService extends FormManagementService {
    createValidatedSelectDataSource(
        searchFunction: (term: string) => Observable<any>,
        validationRules: ValidationRule[],
        buildFlatTree: boolean = false
    ): any {
        const validatedSearchFunction = (term: string) => {
            return searchFunction(term).pipe(
                map(items => items.filter(item => this.validateItem(item, validationRules)))
            );
        };
        
        return this.createSelectDataSource(validatedSearchFunction, buildFlatTree);
    }
    
    private validateItem(item: any, rules: ValidationRule[]): boolean {
        return rules.every(rule => {
            switch (rule.type) {
                case 'required':
                    return item[rule.field] !== null && item[rule.field] !== undefined;
                case 'minLength':
                    return !item[rule.field] || item[rule.field].length >= rule.value;
                case 'maxLength':
                    return !item[rule.field] || item[rule.field].length <= rule.value;
                case 'pattern':
                    return !item[rule.field] || new RegExp(rule.value).test(item[rule.field]);
                default:
                    return true;
            }
        });
    }
}

interface ValidationRule {
    type: 'required' | 'minLength' | 'maxLength' | 'pattern';
    field: string;
    value?: any;
    message?: string;
}
```

---

## 📋 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Use appropriate debounce times
const dataSource = this.formManager.createOptimizedSelectDataSource(
    searchFunction,
    {
        debounceTime: 300,     // Good balance for user experience
        minSearchLength: 2,    // Avoid too many API calls
        maxResults: 50         // Limit results for performance
    }
);

// ✅ Implement caching for frequently used data
const cachedDataSource = this.formManager.createCachedSelectDataSource(
    searchFunction,
    'products',
    false
);
```

### **2. Error Handling**
```typescript
// ✅ Graceful error handling in search functions
const robustSearchFunction = (term: string) => {
    return this.apiService.search(term).pipe(
        catchError(error => {
            console.error('Search failed:', error);
            this.env.showMessage('Search temporarily unavailable', 'warning');
            return of([]); // Return empty array on error
        }),
        timeout(5000), // 5 second timeout
        catchError(() => of([])) // Handle timeout
    );
};
```

### **3. Memory Management**
```typescript
// ✅ Clean up subscriptions
ngOnDestroy() {
    Object.values(this.searchDataSources).forEach(dataSource => {
        if (dataSource.input$) {
            dataSource.input$.complete();
        }
    });
}
```

---

## 🚨 **Security Considerations**

### **1. Input Sanitization**
```typescript
// ✅ Sanitize search terms
private sanitizeSearchTerm(term: string): string {
    return term
        .replace(/[<>]/g, '') // Remove potential HTML
        .replace(/['"]/g, '')  // Remove quotes
        .trim()
        .substring(0, 100);    // Limit length
}
```

### **2. Access Control**
```typescript
// ✅ Check permissions before search
const secureSearchFunction = (term: string) => {
    return this.env.checkFormPermission('/search/products').then(hasPermission => {
        if (!hasPermission) {
            throw new Error('Insufficient permissions');
        }
        
        return this.productService.search(term);
    });
};
```

FormManagementService cung cấp powerful foundation cho dynamic form controls với reactive search, caching, validation, và performance optimization features.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
