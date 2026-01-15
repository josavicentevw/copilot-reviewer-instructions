# Angular Stack-Specific Rules

This document provides Angular-specific validation rules for code reviews. These rules supplement the general checklists and should be applied when reviewing Angular code.

---

## 1. TypeScript Strict Mode Requirements {#typescript-strict}

### Type Safety
- [ ] Enable `strict` mode in `tsconfig.json`
- [ ] No usage of `any` type unless absolutely necessary with justification comment
- [ ] Use proper TypeScript interfaces and types
- [ ] Define interfaces for component inputs/outputs
- [ ] Use type guards for type narrowing

**Examples:**
```typescript
// ❌ BAD - Using any without justification
processData(data: any) {
  return data.value * 2;
}

// ❌ BAD - No interface for component input
@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html'
})
export class UserCardComponent {
  @Input() user: any;
}

// ✅ GOOD - Proper interface definition
export interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
}

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  GUEST = 'GUEST'
}

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html'
})
export class UserCardComponent {
  @Input() user!: User;
  @Output() userClick = new EventEmitter<User>();
}

// ✅ GOOD - Type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  );
}
```

---

## 2. Component Architecture {#component-architecture}

### Component Best Practices
- [ ] Use standalone components (Angular 14+) when appropriate
- [ ] Keep components focused (single responsibility)
- [ ] Use OnPush change detection strategy when possible
- [ ] Implement lifecycle hooks appropriately
- [ ] Unsubscribe from observables in ngOnDestroy

**Examples:**
```typescript
// ❌ BAD - Component doing too much
@Component({
  selector: 'app-user-dashboard',
  templateUrl: './user-dashboard.component.html'
})
export class UserDashboardComponent implements OnInit {
  users: User[] = [];
  
  ngOnInit() {
    // Fetching data directly in component
    this.http.get<User[]>('/api/users').subscribe(users => {
      this.users = users;
    });
  }
}

// ✅ GOOD - Focused component with service injection
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent implements OnInit, OnDestroy {
  users$ = new BehaviorSubject<User[]>([]);
  loading$ = new BehaviorSubject<boolean>(false);
  error$ = new BehaviorSubject<string | null>(null);
  
  private destroy$ = new Subject<void>();
  
  constructor(private userService: UserService) {}
  
  ngOnInit(): void {
    this.loadUsers();
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  private loadUsers(): void {
    this.loading$.next(true);
    this.userService.getUsers()
      .pipe(
        takeUntil(this.destroy$),
        finalize(() => this.loading$.next(false))
      )
      .subscribe({
        next: (users) => this.users$.next(users),
        error: (error) => this.error$.next(error.message)
      });
  }
}

// ✅ GOOD - Standalone component (Angular 14+)
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, MatCardModule],
  templateUrl: './user-card.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user!: User;
  @Output() userSelected = new EventEmitter<User>();
  
  onCardClick(): void {
    this.userSelected.emit(this.user);
  }
}
```

### Smart vs Presentational Components
- [ ] Separate smart (container) and presentational (dumb) components
- [ ] Smart components handle data and state
- [ ] Presentational components only render UI
- [ ] Use @Input/@Output for communication
- [ ] Presentational components should be reusable

**Examples:**
```typescript
// ✅ GOOD - Smart/Container component
@Component({
  selector: 'app-user-container',
  template: `
    <app-user-list
      [users]="users$ | async"
      [loading]="loading$ | async"
      (userSelected)="onUserSelected($event)"
      (deleteUser)="onDeleteUser($event)">
    </app-user-list>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserContainerComponent implements OnInit {
  users$ = this.store.select(selectAllUsers);
  loading$ = this.store.select(selectUsersLoading);
  
  constructor(
    private store: Store,
    private router: Router
  ) {}
  
  ngOnInit(): void {
    this.store.dispatch(loadUsers());
  }
  
  onUserSelected(user: User): void {
    this.router.navigate(['/users', user.id]);
  }
  
  onDeleteUser(user: User): void {
    this.store.dispatch(deleteUser({ userId: user.id }));
  }
}

// ✅ GOOD - Presentational component
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  @Input() users: User[] | null = [];
  @Input() loading = false;
  @Output() userSelected = new EventEmitter<User>();
  @Output() deleteUser = new EventEmitter<User>();
  
  onUserClick(user: User): void {
    this.userSelected.emit(user);
  }
  
  onDeleteClick(user: User): void {
    this.deleteUser.emit(user);
  }
}
```

---

## 3. RxJS and Observables {#rxjs-observables}

### Observable Best Practices
- [ ] Use async pipe in templates (avoid manual subscribe)
- [ ] Unsubscribe from observables properly
- [ ] Use operators for transformation (map, filter, switchMap, etc.)
- [ ] Handle errors with catchError
- [ ] Use subjects sparingly (prefer observables)

**Examples:**
```typescript
// ❌ BAD - Manual subscription without cleanup
export class UserComponent implements OnInit {
  users: User[] = [];
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
    // Missing unsubscribe - memory leak!
  }
}

// ✅ GOOD - Using async pipe
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
}

// ✅ GOOD - Unsubscribe with takeUntil
export class UserComponent implements OnInit, OnDestroy {
  users: User[] = [];
  private destroy$ = new Subject<void>();
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.userService.getUsers()
      .pipe(takeUntil(this.destroy$))
      .subscribe(users => {
        this.users = users;
      });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ GOOD - Complex observable composition
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  searchResults$!: Observable<SearchResult[]>;
  
  ngOnInit() {
    this.searchResults$ = this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(query => query.length >= 3),
      switchMap(query => 
        this.searchService.search(query).pipe(
          catchError(error => {
            console.error('Search failed:', error);
            return of([]);
          })
        )
      ),
      shareReplay(1)
    );
  }
}

// ✅ GOOD - Error handling
export class DataComponent {
  data$ = this.dataService.getData().pipe(
    retry(3),
    catchError(error => {
      this.errorService.handleError(error);
      return of(null);
    })
  );
  
  constructor(
    private dataService: DataService,
    private errorService: ErrorService
  ) {}
}
```

---

## 4. Dependency Injection {#dependency-injection}

### DI Best Practices
- [ ] Use constructor injection
- [ ] Provide services at appropriate level (root, component, module)
- [ ] Use injection tokens for configuration
- [ ] Leverage providedIn: 'root' for singleton services
- [ ] Use factory functions when needed

**Examples:**
```typescript
// ❌ BAD - Direct instantiation
export class UserComponent {
  private userService = new UserService(); // Don't do this!
}

// ✅ GOOD - Constructor injection
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent {
  constructor(
    private userService: UserService,
    private router: Router,
    private dialog: MatDialog
  ) {}
}

// ✅ GOOD - Service provided in root
@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}

// ✅ GOOD - Injection token for configuration
export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

@NgModule({
  providers: [
    { provide: API_BASE_URL, useValue: environment.apiUrl }
  ]
})
export class AppModule {}

// Usage
@Injectable({
  providedIn: 'root'
})
export class ApiService {
  constructor(@Inject(API_BASE_URL) private apiUrl: string) {}
}

// ✅ GOOD - Factory provider
export function createHttpClient(
  apiUrl: string,
  authService: AuthService
): HttpClient {
  // Factory logic
  return new HttpClient(/* ... */);
}

@NgModule({
  providers: [
    {
      provide: HttpClient,
      useFactory: createHttpClient,
      deps: [API_BASE_URL, AuthService]
    }
  ]
})
export class AppModule {}
```

---

## 5. Forms and Validation {#forms-validation}

### Reactive Forms Best Practices
- [ ] Use Reactive Forms for complex forms
- [ ] Define form structure with FormBuilder
- [ ] Implement custom validators when needed
- [ ] Handle form errors in template
- [ ] Use typed forms (Angular 14+)

**Examples:**
```typescript
// ❌ BAD - Template-driven form for complex validation
// (Template-driven forms are fine for simple forms)

// ✅ GOOD - Reactive form with validation
@Component({
  selector: 'app-user-form',
  templateUrl: './user-form.component.html'
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      age: ['', [Validators.required, Validators.min(18), Validators.max(100)]],
      password: ['', [Validators.required, Validators.minLength(8)]],
      confirmPassword: ['', Validators.required]
    }, {
      validators: this.passwordMatchValidator
    });
  }
  
  private passwordMatchValidator(group: FormGroup): ValidationErrors | null {
    const password = group.get('password')?.value;
    const confirmPassword = group.get('confirmPassword')?.value;
    return password === confirmPassword ? null : { passwordMismatch: true };
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      const userData = this.userForm.value;
      // Process form data
    }
  }
}

// ✅ GOOD - Custom validator
export function emailDomainValidator(allowedDomain: string): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const email = control.value;
    if (!email) return null;
    
    const domain = email.split('@')[1];
    return domain === allowedDomain 
      ? null 
      : { invalidDomain: { requiredDomain: allowedDomain, actualDomain: domain } };
  };
}

// Usage
this.userForm = this.fb.group({
  email: ['', [Validators.required, emailDomainValidator('company.com')]]
});

// ✅ GOOD - Typed forms (Angular 14+)
interface UserFormValue {
  name: string;
  email: string;
  age: number;
}

export class UserFormComponent {
  userForm = new FormGroup<{
    name: FormControl<string>;
    email: FormControl<string>;
    age: FormControl<number>;
  }>({
    name: new FormControl('', { nonNullable: true }),
    email: new FormControl('', { nonNullable: true }),
    age: new FormControl(0, { nonNullable: true })
  });
  
  onSubmit() {
    const formValue: UserFormValue = this.userForm.getRawValue();
    // formValue is properly typed
  }
}
```

**Template example:**
```html
<!-- ✅ GOOD - Error handling in template -->
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
  <mat-form-field>
    <mat-label>Name</mat-label>
    <input matInput formControlName="name">
    <mat-error *ngIf="userForm.get('name')?.hasError('required')">
      Name is required
    </mat-error>
    <mat-error *ngIf="userForm.get('name')?.hasError('minlength')">
      Name must be at least 3 characters
    </mat-error>
  </mat-form-field>
  
  <mat-form-field>
    <mat-label>Email</mat-label>
    <input matInput formControlName="email">
    <mat-error *ngIf="userForm.get('email')?.hasError('required')">
      Email is required
    </mat-error>
    <mat-error *ngIf="userForm.get('email')?.hasError('email')">
      Invalid email format
    </mat-error>
  </mat-form-field>
  
  <button mat-raised-button type="submit" [disabled]="!userForm.valid">
    Submit
  </button>
</form>
```

---

## 6. State Management {#state-management}

### NgRx Best Practices (if using)
- [ ] Define actions with proper naming conventions
- [ ] Keep reducers pure functions
- [ ] Use selectors for derived state
- [ ] Handle side effects in effects
- [ ] Use entity adapters for collections

**Examples:**
```typescript
// ✅ GOOD - Actions
export const loadUsers = createAction('[User List] Load Users');
export const loadUsersSuccess = createAction(
  '[User API] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User API] Load Users Failure',
  props<{ error: string }>()
);

// ✅ GOOD - Reducer
export interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  loading: false,
  error: null
};

export const userReducer = createReducer(
  initialState,
  on(loadUsers, (state) => ({ ...state, loading: true, error: null })),
  on(loadUsersSuccess, (state, { users }) => ({ 
    ...state, 
    users, 
    loading: false 
  })),
  on(loadUsersFailure, (state, { error }) => ({ 
    ...state, 
    error, 
    loading: false 
  }))
);

// ✅ GOOD - Selectors
export const selectUserState = (state: AppState) => state.users;
export const selectAllUsers = createSelector(
  selectUserState,
  (state) => state.users
);
export const selectUsersLoading = createSelector(
  selectUserState,
  (state) => state.loading
);
export const selectActiveUsers = createSelector(
  selectAllUsers,
  (users) => users.filter(u => u.isActive)
);

// ✅ GOOD - Effects
@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersFailure({ error: error.message })))
        )
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}

// ✅ GOOD - Using entity adapter
export interface UserEntityState extends EntityState<User> {
  loading: boolean;
  error: string | null;
}

export const userAdapter = createEntityAdapter<User>({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

const initialState: UserEntityState = userAdapter.getInitialState({
  loading: false,
  error: null
});

export const userReducer = createReducer(
  initialState,
  on(loadUsersSuccess, (state, { users }) => 
    userAdapter.setAll(users, { ...state, loading: false })
  )
);

// Selectors
const { selectAll, selectEntities } = userAdapter.getSelectors();
export const selectAllUsers = selectAll;
export const selectUserEntities = selectEntities;
```

---

## 7. HTTP and API Integration {#http-api}

### HTTP Client Best Practices
- [ ] Use HttpClient with typed responses
- [ ] Implement interceptors for common concerns
- [ ] Handle errors globally with interceptors
- [ ] Use environment configuration for API URLs
- [ ] Implement retry logic for failed requests

**Examples:**
```typescript
// ✅ GOOD - Typed HTTP service
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly apiUrl = `${environment.apiUrl}/users`;
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
  
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  createUser(user: CreateUserRequest): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  updateUser(id: string, user: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, user);
  }
  
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}

// ✅ GOOD - Auth interceptor
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }
    
    return next.handle(req);
  }
}

// ✅ GOOD - Error handling interceptor
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(
    private errorService: ErrorService,
    private router: Router
  ) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry(2),
      catchError((error: HttpErrorResponse) => {
        let errorMessage = 'An error occurred';
        
        if (error.error instanceof ErrorEvent) {
          // Client-side error
          errorMessage = `Error: ${error.error.message}`;
        } else {
          // Server-side error
          errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
          
          if (error.status === 401) {
            this.router.navigate(['/login']);
          }
        }
        
        this.errorService.showError(errorMessage);
        return throwError(() => new Error(errorMessage));
      })
    );
  }
}

// Register interceptors
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: ErrorInterceptor, multi: true }
  ]
})
export class AppModule {}
```

---

## 8. Performance Optimization {#performance}

### Performance Best Practices
- [ ] Use OnPush change detection strategy
- [ ] Implement trackBy for *ngFor
- [ ] Lazy load modules
- [ ] Use virtual scrolling for large lists
- [ ] Optimize bundle size with lazy loading

**Examples:**
```typescript
// ✅ GOOD - OnPush change detection
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  @Input() users: User[] = [];
}

// ✅ GOOD - TrackBy function
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngFor="let user of users; trackBy: trackByUserId">
      {{ user.name }}
    </div>
  `
})
export class UserListComponent {
  @Input() users: User[] = [];
  
  trackByUserId(index: number, user: User): string {
    return user.id;
  }
}

// ✅ GOOD - Lazy loaded module
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module').then(m => m.UsersModule)
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];

// ✅ GOOD - Virtual scrolling (Angular CDK)
@Component({
  selector: 'app-large-list',
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackById" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
export class LargeListComponent {
  items: Item[] = [];
  
  trackById(index: number, item: Item): string {
    return item.id;
  }
}
```

---

## Review Checklist Summary

Quick checklist for Angular code reviews:

- [ ] **TypeScript**: Strict mode enabled, proper types, no `any` without justification
- [ ] **Components**: Single responsibility, OnPush strategy, proper lifecycle hooks
- [ ] **RxJS**: Async pipe usage, proper unsubscribe, error handling
- [ ] **DI**: Constructor injection, appropriate service scope
- [ ] **Forms**: Reactive forms for complex validation, custom validators
- [ ] **State**: Proper state management (NgRx if used), immutable updates
- [ ] **HTTP**: Typed responses, interceptors for common concerns
- [ ] **Performance**: TrackBy, lazy loading, virtual scrolling when needed

---

## Tools for Code Quality

**Linters and Formatters:**
```bash
# ESLint with Angular rules
ng lint

# Prettier for formatting
npx prettier --write "src/**/*.{ts,html,scss}"

# Build optimization
ng build --configuration production --optimization
```

**Angular CLI Commands:**
```bash
# Generate component with OnPush
ng generate component user-list --change-detection OnPush

# Generate service
ng generate service services/user

# Generate module with routing
ng generate module users --routing

# Analyze bundle size
ng build --stats-json
npx webpack-bundle-analyzer dist/stats.json
```

---

## References

- [Angular Official Documentation](https://angular.io/docs)
- [Angular Style Guide](https://angular.io/guide/styleguide)
- [RxJS Documentation](https://rxjs.dev/)
- [NgRx Documentation](https://ngrx.io/)
- [Angular Material](https://material.angular.io/)
