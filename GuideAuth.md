# מדריך מקיף: Authorization ו-Permissions עם Roles ב-NestJS

## תוכן עניינים
1. [הקדמה והבנת מושגים](#הקדמה-והבנת-מושגים)
2. [הכנת הפרויקט](#הכנת-הפרויקט)
3. [יצירת Enums ו-DTOs](#יצירת-enums-ו-dtos)
4. [יצירת Auth Module](#יצירת-auth-module)
5. [יצירת Guards להרשאות](#יצירת-guards-להרשאות)
6. [יצירת Decorators מותאמים אישית](#יצירת-decorators-מותאמים-אישית)
7. [יצירת Auth Controller](#יצירת-auth-controller)
8. [יצירת Auth Module](#יצירת-auth-module-1)
9. [דוגמאות שימוש - Products Module](#דוגמאות-שימוש---products-module)
10. [עדכון App Module](#עדכון-app-module)
11. [בדיקת המערכת](#בדיקת-המערכת)
12. [טיפים חשובים](#טיפים-חשובים)
13. [תרגילים לתלמידים](#תרגילים-לתלמידים)

---

## הקדמה והבנת מושגים

### מה ההבדל בין Authentication ל-Authorization?

- **Authentication (אימות)**: "מי אתה?" - בדיקה שהמשתמש הוא מי שהוא אומר שהוא
- **Authorization (הרשאה)**: "מה מותר לך לעשות?" - בדיקה האם למשתמש יש הרשאה לבצע פעולה מסוימת

### מה זה Roles (תפקידים)?

תפקידים הם תוויות שמגדירות את רמת הגישה של משתמש:
- **Admin** - גישה מלאה למערכת
- **Manager** - גישה לניהול תוכן ומשתמשים
- **User** - גישה בסיסית

### ארכיטקטורת המערכת
```
Request → JwtAuthGuard → RolesGuard → Controller → Response
           (בדיקת טוקן)   (בדיקת תפקיד)
```

---

## הכנת הפרויקט

### התקנת חבילות נדרשות
```bash
# יצירת פרויקט חדש
npm i -g @nestjs/cli
nest new roles-authorization-demo
cd roles-authorization-demo

# התקנת חבילות נוספות
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install bcrypt
npm install class-validator class-transformer

# התקנת types
npm install -D @types/passport-jwt @types/bcrypt
```

### מבנה תיקיות הפרויקט
```
src/
├── common/
│   └── enums/
│       └── role.enum.ts
├── auth/
│   ├── decorators/
│   │   ├── roles.decorator.ts
│   │   ├── public.decorator.ts
│   │   └── current-user.decorator.ts
│   ├── guards/
│   │   ├── jwt-auth.guard.ts
│   │   └── roles.guard.ts
│   ├── strategies/
│   │   └── jwt.strategy.ts
│   ├── dto/
│   │   └── register.dto.ts
│   ├── auth.service.ts
│   ├── auth.controller.ts
│   └── auth.module.ts
├── users/
│   └── entities/
│       └── user.entity.ts
├── products/
│   ├── products.controller.ts
│   └── products.module.ts
├── app.module.ts
└── main.ts
```

---

## יצירת Enums ו-DTOs

### 1. יצירת Role Enum

**קובץ:** `src/common/enums/role.enum.ts`
```typescript
// הגדרת התפקידים האפשריים במערכת
export enum Role {
  USER = 'user',      // משתמש רגיל
  MANAGER = 'manager', // מנהל
  ADMIN = 'admin'      // מנהל ראשי
}
```

### 2. יצירת User Entity

**קובץ:** `src/users/entities/user.entity.ts`
```typescript
import { Role } from '../../common/enums/role.enum';

export class User {
  id: number;
  username: string;
  email: string;
  password: string;
  roles: Role[]; // משתמש יכול להיות בעל מספר תפקידים
  createdAt: Date;
  isActive: boolean;
}
```

### 3. יצירת Register DTO

**קובץ:** `src/auth/dto/register.dto.ts`
```typescript
import { IsEmail, IsNotEmpty, IsString, MinLength } from 'class-validator';

export class RegisterDto {
  @IsString()
  @IsNotEmpty({ message: 'שם משתמש הוא שדה חובה' })
  username: string;

  @IsEmail({}, { message: 'אימייל לא תקין' })
  @IsNotEmpty({ message: 'אימייל הוא שדה חובה' })
  email: string;

  @IsString()
  @MinLength(6, { message: 'סיסמה חייבת להכיל לפחות 6 תווים' })
  password: string;
}
```

### 4. יצירת Login DTO

**קובץ:** `src/auth/dto/login.dto.ts`
```typescript
import { IsNotEmpty, IsString } from 'class-validator';

export class LoginDto {
  @IsString()
  @IsNotEmpty({ message: 'שם משתמש הוא שדה חובה' })
  username: string;

  @IsString()
  @IsNotEmpty({ message: 'סיסמה היא שדה חובה' })
  password: string;
}
```

---

## יצירת Auth Module

### 1. Auth Service

**קובץ:** `src/auth/auth.service.ts`
```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import * as bcrypt from 'bcrypt';
import { Role } from '../common/enums/role.enum';
import { User } from '../users/entities/user.entity';

@Injectable()
export class AuthService {
  // דאטהבייס מדומה למשתמשים
  private users: User[] = [
    {
      id: 1,
      username: 'admin',
      email: 'admin@example.com',
      password: '$2b$10$abcdefghijklmnopqrstuv', // סיסמה: admin123
      roles: [Role.ADMIN],
      createdAt: new Date(),
      isActive: true,
    },
    {
      id: 2,
      username: 'manager',
      email: 'manager@example.com',
      password: '$2b$10$abcdefghijklmnopqrstuv', // סיסמה: manager123
      roles: [Role.MANAGER],
      createdAt: new Date(),
      isActive: true,
    },
    {
      id: 3,
      username: 'user',
      email: 'user@example.com',
      password: '$2b$10$abcdefghijklmnopqrstuv', // סיסמה: user123
      roles: [Role.USER],
      createdAt: new Date(),
      isActive: true,
    },
  ];

  constructor(private jwtService: JwtService) {}

  /**
   * רישום משתמש חדש למערכת
   * @param username - שם משתמש
   * @param email - כתובת אימייל
   * @param password - סיסמה (תוצפן אוטומטית)
   * @returns טוקן JWT ופרטי המשתמש
   */
  async register(username: string, email: string, password: string) {
    // בדיקה אם המשתמש כבר קיים
    const existingUser = this.users.find(
      (u) => u.username === username || u.email === email,
    );
    
    if (existingUser) {
      throw new UnauthorizedException('שם משתמש או אימייל כבר קיימים');
    }

    // הצפנת הסיסמה - 10 rounds של salt
    const hashedPassword = await bcrypt.hash(password, 10);

    // יצירת משתמש חדש
    const newUser: User = {
      id: this.users.length + 1,
      username,
      email,
      password: hashedPassword,
      roles: [Role.USER], // כל משתמש חדש מקבל תפקיד USER בלבד
      createdAt: new Date(),
      isActive: true,
    };

    this.users.push(newUser);

    // החזרת טוקן
    return this.generateToken(newUser);
  }

  /**
   * התחברות למערכת
   * @param username - שם משתמש
   * @param password - סיסמה
   * @returns טוקן JWT ופרטי המשתמש
   */
  async login(username: string, password: string) {
    // חיפוש המשתמש
    const user = this.users.find((u) => u.username === username);
    
    if (!user) {
      throw new UnauthorizedException('שם משתמש או סיסמה שגויים');
    }

    // בדיקת הסיסמה מול הסיסמה המוצפנת
    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      throw new UnauthorizedException('שם משתמש או סיסמה שגויים');
    }

    // בדיקה שהמשתמש פעיל
    if (!user.isActive) {
      throw new UnauthorizedException('המשתמש לא פעיל');
    }

    // החזרת טוקן
    return this.generateToken(user);
  }

  /**
   * יצירת JWT Token עם פרטי המשתמש
   * @param user - אובייקט המשתמש
   * @returns אובייקט עם הטוקן ופרטי המשתמש
   */
  private generateToken(user: User) {
    // Payload - המידע שיוצפן בטוקן
    const payload = {
      sub: user.id,              // subject - מזהה המשתמש
      username: user.username,
      email: user.email,
      roles: user.roles,         // חשוב! התפקידים חייבים להיות בטוקן
    };

    return {
      access_token: this.jwtService.sign(payload),
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
        roles: user.roles,
      },
    };
  }

  /**
   * קבלת משתמש לפי ID
   * @param id - מזהה המשתמש
   * @returns אובייקט המשתמש או undefined
   */
  findUserById(id: number): User | undefined {
    return this.users.find((u) => u.id === id);
  }

  /**
   * קבלת כל המשתמשים (ללא סיסמאות)
   * @returns מערך משתמשים ללא שדה הסיסמה
   */
  getAllUsers() {
    return this.users.map(({ password, ...user }) => user);
  }
}
```

### 2. JWT Strategy

**קובץ:** `src/auth/strategies/jwt.strategy.ts`
```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      // מאיפה לשלוף את הטוקן - מה-Authorization header
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      
      // לא להתעלם מטוקנים שפגו
      ignoreExpiration: false,
      
      // המפתח הסודי לאימות הטוקן
      // במציאות: להשתמש ב-ConfigService ו-environment variable
      secretOrKey: 'YOUR_SECRET_KEY_HERE',
    });
  }

  /**
   * פונקציה זו רצה אוטומטית אחרי שהטוקן עבר אימות
   * מה שמוחזר כאן יהיה זמין ב-request.user בכל ה-controllers
   * @param payload - המידע שהוצפן בטוקן
   * @returns אובייקט המשתמש שיהיה זמין ב-request
   */
  async validate(payload: any) {
    // בדיקה נוספת - אפשר לבדוק אם המשתמש עדיין קיים בדאטהבייס
    if (!payload) {
      throw new UnauthorizedException('טוקן לא תקין');
    }

    return {
      userId: payload.sub,
      username: payload.username,
      email: payload.email,
      roles: payload.roles, // חשוב! מעבירים את התפקידים הלאה
    };
  }
}
```

---

## יצירת Guards להרשאות

### 1. JWT Auth Guard

**קובץ:** `src/auth/guards/jwt-auth.guard.ts`
```typescript
import { Injectable, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { AuthGuard } from '@nestjs/passport';
import { Observable } from 'rxjs';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

/**
 * Guard זה בודק שהמשתמש מחובר (יש לו טוקן תקף)
 * הוא פועל על כל הנתיבים אלא אם מסומנים עם @Public()
 */
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    // בדיקה אם הנתיב מסומן כציבורי
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    // אם הנתיב ציבורי - מאפשרים גישה ללא בדיקת טוקן
    if (isPublic) {
      return true;
    }

    // אחרת - בודקים טוקן
    return super.canActivate(context);
  }

  /**
   * פונקציה זו מופעלת כאשר האימות נכשל
   */
  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('נדרש אימות - אנא התחבר למערכת');
    }
    return user;
  }
}
```

### 2. Roles Guard

**קובץ:** `src/auth/guards/roles.guard.ts`
```typescript
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Role } from '../../common/enums/role.enum';
import { ROLES_KEY } from '../decorators/roles.decorator';

/**
 * Guard זה בודק שלמשתמש יש את התפקיד הנדרש
 * הוא פועל רק על נתיבים שמסומנים עם @Roles()
 */
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // שליפת התפקידים הנדרשים מה-metadata
    // בודק גם ברמת הפונקציה וגם ברמת הקלאס
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(), // @Roles() על הפונקציה
      context.getClass(),   // @Roles() על הקלאס
    ]);

    // אם אין תפקידים נדרשים - מאפשרים גישה
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    // שליפת המשתמש מה-request (הוכנס על ידי JwtAuthGuard)
    const { user } = context.switchToHttp().getRequest();

    // אם אין משתמש (לא אמור לקרות אם JwtAuthGuard עובד)
    if (!user) {
      throw new ForbiddenException('לא ניתן לאמת את זהות המשתמש');
    }

    // בדיקה אם למשתמש יש לפחות אחד מהתפקידים הנדרשים
    const hasRole = requiredRoles.some((role) => user.roles?.includes(role));

    if (!hasRole) {
      throw new ForbiddenException(
        `נדרש אחד מהתפקידים הבאים: ${requiredRoles.join(', ')}`
      );
    }

    return true;
  }
}
```

---

## יצירת Decorators מותאמים אישית

### 1. Roles Decorator

**קובץ:** `src/auth/decorators/roles.decorator.ts`
```typescript
import { SetMetadata } from '@nestjs/common';
import { Role } from '../../common/enums/role.enum';

export const ROLES_KEY = 'roles';

/**
 * Decorator זה מאפשר להגדיר אילו תפקידים נדרשים לגישה לנתיב
 * שימוש: @Roles(Role.ADMIN, Role.MANAGER)
 */
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);
```

**דוגמאות שימוש:**
```typescript
// רק ADMIN יכול לגשת
@Roles(Role.ADMIN)

// ADMIN או MANAGER יכולים לגשת
@Roles(Role.ADMIN, Role.MANAGER)

// כל התפקידים יכולים לגשת
@Roles(Role.ADMIN, Role.MANAGER, Role.USER)
```

### 2. Public Decorator

**קובץ:** `src/auth/decorators/public.decorator.ts`
```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';

/**
 * Decorator זה מסמן נתיבים שלא צריכים אימות
 * נתיבים כאלה פתוחים לכולם (login, register, וכו')
 * שימוש: @Public()
 */
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

**דוגמאות שימוש:**
```typescript
@Public()
@Post('login')
async login() {
  // נתיב זה פתוח לכולם ללא אימות
}
```

### 3. Current User Decorator

**קובץ:** `src/auth/decorators/current-user.decorator.ts`
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

/**
 * Decorator זה מאפשר לשלוף את המשתמש הנוכחי בקלות
 * במקום לעשות: const user = request.user
 * אפשר לעשות: @CurrentUser() user
 */
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

**דוגמאות שימוש:**
```typescript
@Get('profile')
getProfile(@CurrentUser() user: any) {
  // user כבר מוכן לשימוש
  return {
    username: user.username,
    roles: user.roles,
  };
}
```

---

## יצירת Auth Controller

**קובץ:** `src/auth/auth.controller.ts`
```typescript
import { 
  Body, 
  Controller, 
  Post, 
  Get,
  ValidationPipe,
  UseGuards 
} from '@nestjs/common';
import { AuthService } from './auth.service';
import { RegisterDto } from './dto/register.dto';
import { LoginDto } from './dto/login.dto';
import { Public } from './decorators/public.decorator';
import { CurrentUser } from './decorators/current-user.decorator';
import { Roles } from './decorators/roles.decorator';
import { RolesGuard } from './guards/roles.guard';
import { Role } from '../common/enums/role.enum';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  /**
   * רישום משתמש חדש
   * נתיב ציבורי - לא צריך אימות
   */
  @Public()
  @Post('register')
  async register(@Body(ValidationPipe) registerDto: RegisterDto) {
    return this.authService.register(
      registerDto.username,
      registerDto.email,
      registerDto.password,
    );
  }

  /**
   * התחברות למערכת
   * נתיב ציבורי - לא צריך אימות
   */
  @Public()
  @Post('login')
  async login(@Body(ValidationPipe) loginDto: LoginDto) {
    return this.authService.login(loginDto.username, loginDto.password);
  }

  /**
   * קבלת פרטי המשתמש המחובר
   * נתיב מוגן - דורש אימות
   */
  @Get('profile')
  getProfile(@CurrentUser() user: any) {
    return {
      message: 'פרופיל המשתמש',
      user: {
        id: user.userId,
        username: user.username,
        email: user.email,
        roles: user.roles,
      },
    };
  }

  /**
   * קבלת כל המשתמשים
   * רק ADMIN יכול לראות
   */
  @Get('users')
  @UseGuards(RolesGuard)
  @Roles(Role.ADMIN)
  getAllUsers(@CurrentUser() user: any) {
    return {
      message: 'רשימת כל המשתמשים',
      requestedBy: user.username,
      users: this.authService.getAllUsers(),
    };
  }
}
```

---

## יצירת Auth Module

**קובץ:** `src/auth/auth.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Module({
  imports: [
    // הגדרת Passport
    PassportModule,
    
    // הגדרת JWT
    JwtModule.register({
      secret: 'YOUR_SECRET_KEY_HERE', // במציאות: ConfigService
      signOptions: { 
        expiresIn: '24h', // הטוקן תקף ל-24 שעות
      },
    }),
  ],
  controllers: [AuthController],
  providers: [
    AuthService,
    JwtStrategy,
    
    // הפיכת JwtAuthGuard ל-global
    // כל הנתיבים יהיו מוגנים אוטומטית אלא אם מסומנים ב-@Public()
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
  ],
  exports: [AuthService],
})
export class AuthModule {}
```

---

## דוגמאות שימוש - Products Module

### Products Controller

**קובץ:** `src/products/products.controller.ts`
```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Body, 
  Param, 
  UseGuards,
  HttpCode,
  HttpStatus 
} from '@nestjs/common';
import { Role } from '../common/enums/role.enum';
import { Roles } from '../auth/decorators/roles.decorator';
import { RolesGuard } from '../auth/guards/roles.guard';
import { CurrentUser } from '../auth/decorators/current-user.decorator';

@Controller('products')
@UseGuards(RolesGuard) // הפעלת בדיקת תפקידים על כל הנתיבים
export class ProductsController {
  
  /**
   * קבלת כל המוצרים
   * כל משתמש מחובר יכול לצפות במוצרים
   */
  @Get()
  findAll(@CurrentUser() user: any) {
    return {
      message: 'רשימת כל המוצרים',
      requestedBy: user.username,
      userRoles: user.roles,
      products: [
        { id: 1, name: 'מחשב נייד', price: 3000, category: 'אלקטרוניקה' },
        { id: 2, name: 'עכבר', price: 50, category: 'אלקטרוניקה' },
        { id: 3, name: 'מקלדת', price: 150, category: 'אלקטרוניקה' },
      ],
    };
  }

  /**
   * קבלת מוצר בודד לפי ID
   * כל משתמש מחובר יכול לצפות במוצר
   */
  @Get(':id')
  findOne(@Param('id') id: string, @CurrentUser() user: any) {
    return {
      message: `פרטי מוצר ${id}`,
      requestedBy: user.username,
      product: { 
        id: parseInt(id), 
        name: 'מחשב נייד', 
        price: 3000,
        description: 'מחשב נייד חזק ומהיר',
        stock: 15
      },
    };
  }

  /**
   * יצירת מוצר חדש
   * רק MANAGER ו-ADMIN יכולים ליצור מוצרים
   */
  @Post()
  @Roles(Role.MANAGER, Role.ADMIN)
  @HttpCode(HttpStatus.CREATED)
  create(@Body() body: any, @CurrentUser() user: any) {
    return {
      message: 'מוצר נוצר בהצלחה',
      createdBy: user.username,
      userRoles: user.roles,
      product: {
        id: Date.now(), // ID זמני
        ...body,
        createdAt: new Date(),
      },
    };
  }

  /**
   * עדכון מוצר קיים
   * רק MANAGER ו-ADMIN יכולים לעדכן מוצרים
   */
  @Put(':id')
  @Roles(Role.MANAGER, Role.ADMIN)
  update(@Param('id') id: string, @Body() body: any, @CurrentUser() user: any) {
    return {
      message: `מוצר ${id} עודכן בהצלחה`,
      updatedBy: user.username,
      userRoles: user.roles,
      product: {
        id: parseInt(id),
        ...body,
        updatedAt: new Date(),
      },
    };
  }

  /**
   * מחיקת מוצר
   * רק ADMIN יכול למחוק מוצרים
   */
  @Delete(':id')
  @Roles(Role.ADMIN)
  @HttpCode(HttpStatus.OK)
  delete(@Param('id') id: string, @CurrentUser() user: any) {
    return {
      message: `מוצר ${id} נמחק בהצלחה`,
      deletedBy: user.username,
      userRoles: user.roles,
      deletedAt: new Date(),
    };
  }

  /**
   * קבלת סטטיסטיקות מוצרים
   * רק ADMIN ו-MANAGER יכולים לראות סטטיסטיקות
   */
  @Get('stats/overview')
  @Roles(Role.ADMIN, Role.MANAGER)
  getStats(@CurrentUser() user: any) {
    return {
      message: 'סטטיסטיקות מוצרים',
      requestedBy: user.username,
      stats: {
        totalProducts: 150,
        totalValue: 450000,
        lowStock: 12,
        outOfStock: 3,
      },
    };
  }
}
```

### Products Module

**קובץ:** `src/products/products.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { ProductsController } from './products.controller';

@Module({
  controllers: [ProductsController],
  providers: [],
})
export class ProductsModule {}
```

---

## עדכון App Module

**קובץ:** `src/app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { AuthModule } from './auth/auth.module';
import { ProductsModule } from './products/products.module';

@Module({
  imports: [
    AuthModule,      // מודול האימות וההרשאות
    ProductsModule,  // מודול המוצרים
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

---

## בדיקת המערכת

### 1. הרצת השרת
```bash
npm run start:dev
```

השרת יעלה על: `http://localhost:3000`

### 2. בדיקות עם Postman/Thunder Client

#### א. רישום משתמש חדש
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "username": "testuser",
  "email": "test@example.com",
  "password": "test123456"
}
```

**תשובה מצופה:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 4,
    "username": "testuser",
    "email": "test@example.com",
    "roles": ["user"]
  }
}
```

#### ב. התחברות (USER)
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "username": "user",
  "password": "user123"
}
```

#### ג. התחברות (ADMIN)
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "admin123"
}
```

#### ד. צפייה בפרופיל (דורש אימות)
```http
GET http://localhost:3000/auth/profile
Authorization: Bearer YOUR_TOKEN_HERE
```

#### ה. צפייה במוצרים (כל משתמש מחובר)
```http
GET http://localhost:3000/products
Authorization: Bearer YOUR_USER_TOKEN
```

**תשובה מצופה:**
```json
{
  "message": "רשימת כל המוצרים",
  "requestedBy": "user",
  "userRoles": ["user"],
  "products": [
    { "id": 1, "name": "מחשב נייד", "price": 3000, "category": "אלקטרוניקה" },
    { "id": 2, "name": "עכבר", "price": 50, "category": "אלקטרוניקה" }
  ]
}
```

#### ו. יצירת מוצר (רק MANAGER/ADMIN)
```http
POST http://localhost:3000/products
Authorization: Bearer YOUR_ADMIN_TOKEN
Content-Type: application/json

{
  "name": "מקלדת מכנית",
  "price": 250,
  "category": "אלקטרוניקה",
  "stock": 30
}
```

**אם ננסה עם USER Token - נקבל שגיאה:**
```json
{
  "statusCode": 403,
  "message": "נדרש אחד מהתפקידים הבאים: manager, admin",
  "error": "Forbidden"
}
```

#### ז. מחיקת מוצר (רק ADMIN)
```http
DELETE http://localhost:3000/products/1
Authorization: Bearer YOUR_ADMIN_TOKEN
```

**אם ננסה עם MANAGER Token - נקבל שגיאה:**
```json
{
  "statusCode": 403,
  "message": "נדרש אחד מהתפקידים הבאים: admin",
  "error": "Forbidden"
}
```

#### ח. צפייה בכל המשתמשים (רק ADMIN)
```http
GET http://localhost:3000/auth/users
Authorization: Bearer YOUR_ADMIN_TOKEN
```

---

## טבלת סיכום הרשאות

| פעולה | נתיב | USER | MANAGER | ADMIN |
|-------|------|------|---------|-------|
| רישום | POST /auth/register | ✅ (ללא אימות) | ✅ (ללא אימות) | ✅ (ללא אימות) |
| התחברות | POST /auth/login | ✅ (ללא אימות) | ✅ (ללא אימות) | ✅ (ללא אימות) |
| פרופיל | GET /auth/profile | ✅ | ✅ | ✅ |
| רשימת משתמשים | GET /auth/users | ❌ | ❌ | ✅ |
| צפייה במוצרים | GET /products | ✅ | ✅ | ✅ |
| צפייה במוצר בודד | GET /products/:id | ✅ | ✅ | ✅ |
| יצירת מוצר | POST /products | ❌ | ✅ | ✅ |
| עדכון מוצר | PUT /products/:id | ❌ | ✅ | ✅ |
| מחיקת מוצר | DELETE /products/:id | ❌ | ❌ | ✅ |
| סטטיסטיקות | GET /products/stats | ❌ | ✅ | ✅ |

---

## טיפים חשובים

### 1. אבטחה
```typescript
// ❌ לא טוב - מפתח קבוע בקוד
JwtModule.register({
  secret: 'my-secret-key',
})

// ✅ טוב - משתמש ב-environment variables
JwtModule.registerAsync({
  useFactory: (configService: ConfigService) => ({
    secret: configService.get<string>('JWT_SECRET'),
    signOptions: { expiresIn: '24h' },
  }),
  inject: [ConfigService],
})
```

### 2. סיסמאות
```typescript
// ✅ תמיד הצפן סיסמאות
const hashedPassword = await bcrypt.hash(password, 10);

// ✅ תמיד השווה עם bcrypt.compare
const isValid = await bcrypt.compare(password, user.password);
```

### 3. טוקנים
```typescript
// הגדר זמן תפוגה סביר
signOptions: { 
  expiresIn: '24h'  // יום אחד
  // expiresIn: '7d'   // שבוע
  // expiresIn: '15m'  // 15 דקות
}
```

### 4. בדיקות

תמיד בדוק:
- ✅ משתמש לא מחובר מנסה לגשת לנתיב מוגן
- ✅ משתמש עם תפקיד לא מתאים מנסה לבצע פעולה
- ✅ טוקן לא תקף
- ✅ טוקן שפג תוקפו
- ✅ משתמש לא פעיל

### 5. הודעות שגיאה
```typescript
// ✅ טוב - הודעה ברורה
throw new UnauthorizedException('נדרש אימות - אנא התחבר למערכת');
throw new ForbiddenException('אין לך הרשאה לבצע פעולה זו');

// ❌ לא טוב - הודעה לא ברורה
throw new UnauthorizedException('Error');
```

---

## תרגילים לתלמידים

### תרגיל 1: הוספת תפקיד SUPER_ADMIN
צור תפקיד `SUPER_ADMIN` שיכול:
- לראות לוג של כל הפעולות במערכת
- לשנות תפקידים של משתמשים אחרים
- לחסום/לשחרר משתמשים

**רמז:**
```typescript
enum Role {
  USER = 'user',
  MANAGER = 'manager',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
}
```

### תרגיל 2: ניהול תפקידי משתמשים
צור endpoint שמאפשר ל-ADMIN להוסיף/להסיר תפקידים למשתמשים:
```typescript
@Put('users/:id/roles')
@Roles(Role.ADMIN)
updateUserRoles(
  @Param('id') userId: string,
  @Body() body: { roles: Role[] }
) {
  // הטמעה שלך כאן
}
```

### תרגיל 3: מניעת מחיקה עצמית
הוסף בדיקה שמונעת ממשתמש למחוק את עצמו:
```typescript
@Delete('users/:id')
@Roles(Role.ADMIN)
deleteUser(
  @Param('id') userId: string,
  @CurrentUser() currentUser: any
) {
  // בדוק שהמשתמש לא מוחק את עצמו
}
```

### תרגיל 4: MinimumRole Decorator
צור decorator חדש `@MinimumRole()` שמאפשר גישה מתפקיד מסוים ומעלה:
```typescript
// דוגמת שימוש:
@MinimumRole(Role.MANAGER)
// אם המשתמש הוא MANAGER או ADMIN - מאפשר גישה
// אם המשתמש הוא USER - חוסם גישה
```

**רמז:** השתמש בהיררכיה:
```typescript
const roleHierarchy = {
  [Role.USER]: 0,
  [Role.MANAGER]: 1,
  [Role.ADMIN]: 2,
  [Role.SUPER_ADMIN]: 3,
};
```

### תרגיל 5: לוג פעילויות
צור interceptor שמתעד כל פעולה במערכת:
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    // תעד: מי, מתי, מה, איזה endpoint
    return next.handle();
  }
}
```

### תרגיל 6: הגבלת קצב (Rate Limiting)
צור guard שמגביל כמה פעולות משתמש יכול לבצע בדקה:
```typescript
@Injectable()
export class RateLimitGuard implements CanActivate {
  // הגבל ל-10 requests לדקה למשתמש
}
```

---

## משאבים נוספים

### תיעוד רשמי
- [NestJS Authentication](https://docs.nestjs.com/security/authentication)
- [NestJS Authorization](https://docs.nestjs.com/security/authorization)
- [Passport JWT](http://www.passportjs.org/packages/passport-jwt/)

### מושגים חשובים
- **JWT (JSON Web Token)**: טוקן מוצפן שמכיל מידע על המשתמש
- **Guard**: מנגנון שבודק אם request יכול להמשיך
- **Decorator**: פונקציה שמוסיפה metadata או מתנהגות לקוד
- **Middleware**: פונקציה שרצה לפני route handler
- **Interceptor**: מנגנון שיכול לשנות request או response

### דוגמת Environment Variables

צור קובץ `.env`:
```env
JWT_SECRET=your-super-secret-key-change-this-in-production
JWT_EXPIRES_IN=24h
PORT=3000
```

---

## סיכום

במדריך זה למדנו:

1. ✅ הבדל בין Authentication ל-Authorization
2. ✅ שימוש ב-JWT Tokens
3. ✅ יצירת מערכת תפקידים (Roles)
4. ✅ בניית Guards לבדיקת הרשאות
5. ✅ יצירת Decorators מותאמים אישית
6. ✅ הגנה על נתיבים לפי תפקידים
7. ✅ בדיקות מקיפות של המערכת

**זכור:**
- תמיד הצפן סיסמאות
- השתמש ב-environment variables
- בדוק כל תרחיש אפשרי
- החזר הודעות שגיאה ברורות
- שמור על קוד נקי וקריא

בהצלחה! 🚀