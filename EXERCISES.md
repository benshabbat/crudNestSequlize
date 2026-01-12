# מדריך תרגילים - NestJS + Sequelize + Docker Compose

## 📚 מבוא
מדריך זה מכיל תרגילים מעשיים לתלמידים הלומדים NestJS עם Sequelize ו-Docker Compose.
התרגילים מסודרים בהדרגה מהפשוט למורכב, ומבוססים על פרויקטים אמיתיים.

---

## 🎯 שלב 1: הכרת NestJS והקמת הפרויקט

### תרגיל 1.1: הקמת פרויקט NestJS
**מטרה**: ללמוד את מבנה הפרויקט הבסיסי של NestJS

**משימות**:
1. התקן את NestJS CLI באופן גלובלי
2. צור פרויקט NestJS חדש בשם `student-management`
3. הרץ את השרת ווודא שהוא עובד על פורט 3000
4. בדוק את המבנה של הקבצים שנוצרו והבן את תפקידם

**שאלות להבנה**:
- מהו תפקידו של קובץ `main.ts`?
- מה ההבדל בין Controller ל-Service?
- מה המשמעות של `@Module` decorator?

---

### תרגיל 1.2: יצירת Controller בסיסי
**מטרה**: להבין איך ליצור endpoints בסיסיים

**משימות**:
1. צור controller חדש בשם `students`
2. הוסף route שמחזיר רשימה של שמות תלמידים (מערך סטטי)
3. הוסף route שמקבל ID ומחזיר תלמיד ספציפי
4. הוסף route שמקבל שם בגוף הבקשה ומחזיר הודעת ברכה

**דרישות**:
- השתמש ב-decorators המתאימים: `@Get`, `@Post`, `@Param`, `@Body`
- החזר status codes מתאימים

---

### תרגיל 1.3: יצירת Service
**מטרה**: להפריד את הלוגיקה העסקית מה-Controller

**משימות**:
1. צור service בשם `students`
2. העבר את הלוגיקה של ניהול התלמידים מה-Controller ל-Service
3. הזרק את ה-Service ל-Controller באמצעות Dependency Injection
4. הוסף פונקציות: `getAllStudents`, `getStudentById`, `addStudent`

**דרישות**:
- השתמש ב-`@Injectable` decorator
- ודא שה-Service מספק את כל הפונקציונליות הדרושה

---

## 🗄️ שלב 2: הוספת Sequelize לפרויקט

### תרגיל 2.1: הגדרת Sequelize
**מטרה**: להתחבר למסד נתונים PostgreSQL באמצעות Sequelize

**משימות**:
1. התקן את החבילות הנדרשות: `@nestjs/sequelize`, `sequelize`, `sequelize-typescript`, `pg`, `pg-hstore`
2. הוסף את `SequelizeModule` ל-`AppModule`
3. הגדר חיבור למסד נתונים עם הפרמטרים הבאים:
   - host: localhost
   - port: 5432
   - database: students_db
   - username: postgres
   - password: postgres
4. הגדר `autoLoadModels: true` ו-`synchronize: true`

**שאלות להבנה**:
- מה המשמעות של `synchronize: true`?
- מדוע לא מומלץ להשתמש ב-`synchronize` בסביבת production?

---

### תרגיל 2.2: יצירת Model ראשון
**מטרה**: ליצור מודל של Student עם Sequelize

**משימות**:
1. צור תיקיה `models` בתוך תיקיית `students`
2. צור מודל `Student` עם השדות הבאים:
   - `id` (מספר, primary key, auto increment)
   - `firstName` (מחרוזת, חובה)
   - `lastName` (מחרוזת, חובה)
   - `email` (מחרוזת, ייחודי, חובה)
   - `age` (מספר)
   - `createdAt` (תאריך)
   - `updatedAt` (תאריך)
3. רשום את המודל ב-`StudentsModule`

**דרישות**:
- השתמש ב-decorators: `@Table`, `@Column`, `@PrimaryKey`, `@AutoIncrement`
- הגדר validations מתאימים

---

### תרגיל 2.3: CRUD בסיסי עם Sequelize
**מטרה**: לממש פעולות CRUD מלאות

**משימות**:
1. עדכן את ה-Service לעבוד עם Sequelize במקום מערך סטטי
2. מימוש הפונקציות הבאות:
   - `findAll()` - קבלת כל התלמידים
   - `findOne(id)` - קבלת תלמיד לפי ID
   - `create(studentData)` - יצירת תלמיד חדש
   - `update(id, studentData)` - עדכון תלמיד קיים
   - `remove(id)` - מחיקת תלמיד
3. עדכן את ה-Controller עם כל ה-endpoints המתאימים

**דרישות**:
- טיפול בשגיאות (תלמיד לא נמצא, validation errors)
- החזרת status codes מתאימים (200, 201, 404, וכו')

---

## 🔍 שלב 3: DTO ו-Validation

### תרגיל 3.1: יצירת DTOs
**מטרה**: להשתמש ב-DTOs לבדיקת נתונים

**משימות**:
1. התקן את החבילות: `class-validator` ו-`class-transformer`
2. הפעל את `ValidationPipe` באופן גלובלי
3. צור את ה-DTOs הבאים:
   - `CreateStudentDto` - לבדיקת נתוני תלמיד חדש
   - `UpdateStudentDto` - לבדיקת עדכון תלמיד
4. הוסף validations:
   - שם פרטי ומשפחה - מינימום 2 תווים
   - אימייל - פורמט תקין
   - גיל - בין 5 ל-120

**דרישות**:
- השתמש ב-decorators: `@IsString`, `@IsEmail`, `@IsNumber`, `@Min`, `@Max`, `@MinLength`
- הוסף הודעות שגיאה בעברית

---

### תרגיל 3.2: Custom Validators
**מטרה**: ליצור validators מותאמים אישית

**משימות**:
1. צור validator שבודק שאימייל מסתיים ב-`@student.com`
2. צור validator שבודק שהשם לא מכיל מספרים
3. צור validator שבודק שהגיל הוא מספר שלם
4. הוסף את ה-validators ל-DTOs

**שאלות להבנה**:
- מתי כדאי להשתמש ב-custom validator?
- איך מטמיעים validator אסינכרוני?

---

## 🔗 שלב 4: יחסים בין טבלאות (Relations)

### תרגיל 4.1: יצירת מודל Course
**מטרה**: להבין יחסים של One-to-Many

**משימות**:
1. צור מודול, controller, service ומודל חדשים עבור `Course`
2. מודל Course צריך להכיל:
   - `id`, `name`, `description`, `credits`, `teacherName`
3. הגדר יחס One-to-Many בין Course ל-Student:
   - קורס אחד יכול להכיל תלמידים רבים
   - תלמיד שייך לקורס אחד
4. הוסף `courseId` למודל Student

**דרישות**:
- השתמש ב-decorators: `@HasMany`, `@BelongsTo`, `@ForeignKey`
- בדוק את היחס עם queries

---

### תרגיל 4.2: Many-to-Many Relationship
**מטרה**: להבין יחסי Many-to-Many

**משימות**:
1. שנה את היחס: תלמיד יכול להירשם למספר קורסים וקורס יכול להכיל מספר תלמידים
2. צור טבלת ביניים `StudentCourses` עם:
   - `studentId`, `courseId`, `enrollmentDate`, `grade`
3. עדכן את המודלים עם `@BelongsToMany`
4. צור endpoints לרישום תלמיד לקורס ולהסרתו

**דרישות**:
- השתמש ב-`through` option
- הוסף אפשרות לשלוף תלמיד עם כל הקורסים שלו

---

### תרגיל 4.3: Eager vs Lazy Loading
**מטרה**: להבין את ההבדלים בין סוגי הטעינה

**משימות**:
1. צור endpoint שמחזיר תלמיד עם כל הקורסים שלו (Eager Loading)
2. צור endpoint שמחזיר תלמיד בלבד (Lazy Loading)
3. צור endpoint שמחזיר קורס עם כל התלמידים שלו
4. השווה את ביצועי ה-queries

**שאלות להבנה**:
- מתי כדאי להשתמש ב-Eager Loading?
- מה החסרונות של Eager Loading?

---

## 🐳 שלב 5: Docker ו-Docker Compose

### תרגיל 5.1: Dockerize של אפליקציית NestJS
**מטרה**: ליצור Docker image לאפליקציה

**משימות**:
1. צור `Dockerfile` לאפליקציית NestJS
2. הקובץ צריך:
   - להשתמש ב-Node.js image
   - להעתיק את הקבצים
   - להתקין dependencies
   - לבנות את האפליקציה
   - להריץ את האפליקציה
3. בנה את ה-image
4. הרץ container מה-image ובדוק שהוא עובד

**דרישות**:
- השתמש ב-multi-stage build לאופטימיזציה
- חשוף את הפורט הנכון

---

### תרגיל 5.2: Docker Compose - PostgreSQL
**מטרה**: להריץ PostgreSQL ב-container

**משימות**:
1. צור קובץ `docker-compose.yml`
2. הגדר service של PostgreSQL עם:
   - שם: postgres
   - image: postgres:15
   - environment variables (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB)
   - volume למידע מתמשך
   - port mapping
3. הרץ את ה-container
4. התחבר למסד הנתונים ובדוק שהוא עובד

**דרישות**:
- השתמש ב-named volume
- הגדר restart policy

---

### תרגיל 5.3: Docker Compose - Full Stack
**מטרה**: להריץ את כל האפליקציה עם Docker Compose

**משימות**:
1. עדכן את `docker-compose.yml` להכיל:
   - PostgreSQL service
   - NestJS application service
2. הגדר תלות: האפליקציה צריכה להמתין ל-PostgreSQL
3. הגדר network משותף
4. עדכן את הגדרות החיבור למסד נתונים להשתמש במשתני סביבה
5. הרץ את כל המערכת עם `docker-compose up`

**דרישות**:
- השתמש ב-`depends_on`
- הגדר `.env` file למשתני סביבה
- בדוק שהכל עובד ביחד

---

## 🔐 שלב 6: Authentication ו-Authorization - מדריך מפורט למתחילים

---

## 📖 **מדריך מלא: Authorization ו-Permissions עם Roles**

### 🎯 מה נלמד במדריך זה?
במדריך זה נלמד איך ליצור מערכת הרשאות מלאה שכוללת:
1. **Authentication** (זיהוי משתמשים) - מי אתה?
2. **Authorization** (הרשאות) - מה אתה יכול לעשות?
3. **Roles** (תפקידים) - איזה תפקיד יש לך במערכת?
4. **Permissions** (הרשאות ספציפיות) - הרשאות מפורטות לכל פעולה

---

## 🏗️ **שלב 1: הבנת המושגים הבסיסיים**

### מה זה Authentication?
**Authentication** = זיהוי משתמש
- בודק אם המשתמש הוא באמת מי שהוא טוען שהוא
- דוגמה: התחברות עם שם משתמש וסיסמה

### מה זה Authorization?
**Authorization** = הרשאות גישה
- בודק אם למשתמש יש הרשאה לבצע פעולה מסוימת
- דוגמה: רק מנהל יכול למחוק משתמשים

### מה זה Roles?
**Roles** = תפקידים במערכת
- קבוצה של הרשאות שמוגדרות לפי תפקיד
- דוגמאות: תלמיד, מורה, מנהל

### מה זה Permissions?
**Permissions** = הרשאות ספציפיות
- הרשאות מפורטות לפעולות ספציפיות
- דוגמאות: `READ_STUDENTS`, `CREATE_COURSE`, `DELETE_USER`

---

## 📦 **שלב 2: התקנת החבילות הנדרשות**

### 2.1: התקנת חבילות Authentication
פתח טרמינל והרץ:
```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install @types/passport-jwt --save-dev
npm install bcrypt
npm install @types/bcrypt --save-dev
```

### 2.2: הסבר על החבילות
- **@nestjs/jwt**: מודול של NestJS לעבודה עם JWT tokens
- **@nestjs/passport**: אינטגרציה של Passport ב-NestJS
- **passport**: ספריית authentication פופולרית ל-Node.js
- **passport-jwt**: אסטרטגיה של JWT ל-Passport
- **bcrypt**: להצפנת סיסמאות

---

## 👤 **שלב 3: יצירת מודול המשתמשים (Users)**

### 3.1: יצירת המבנה
הרץ את הפקודות הבאות:
```bash
nest g module users
nest g controller users
nest g service users
```

### 3.2: יצירת מודל User
צור קובץ: `src/users/models/user.model.ts`

```typescript
import {
  Table,
  Column,
  Model,
  DataType,
  PrimaryKey,
  AutoIncrement,
  Unique,
  AllowNull,
  Default,
} from 'sequelize-typescript';

// הגדרת enum לתפקידים
export enum UserRole {
  ADMIN = 'admin',
  TEACHER = 'teacher',
  STUDENT = 'student',
}

@Table({
  tableName: 'users',
  timestamps: true,
})
export class User extends Model {
  @PrimaryKey
  @AutoIncrement
  @Column(DataType.INTEGER)
  id: number;

  @AllowNull(false)
  @Column(DataType.STRING)
  firstName: string;

  @AllowNull(false)
  @Column(DataType.STRING)
  lastName: string;

  @Unique
  @AllowNull(false)
  @Column(DataType.STRING)
  username: string;

  @Unique
  @AllowNull(false)
  @Column(DataType.STRING)
  email: string;

  @AllowNull(false)
  @Column(DataType.STRING)
  password: string;

  @Default(UserRole.STUDENT)
  @Column(DataType.ENUM(...Object.values(UserRole)))
  role: UserRole;

  @Default(true)
  @Column(DataType.BOOLEAN)
  isActive: boolean;
}
```

**הסבר על השדות**:
- `id`: מזהה ייחודי
- `firstName`, `lastName`: שם המשתמש
- `username`: שם משתמש ייחודי
- `email`: אימייל ייחודי
- `password`: סיסמה מוצפנת
- `role`: תפקיד המשתמש (תלמיד/מורה/מנהל)
- `isActive`: האם המשתמש פעיל

### 3.3: יצירת DTOs
צור קובץ: `src/users/dto/create-user.dto.ts`

```typescript
import {
  IsString,
  IsEmail,
  IsEnum,
  MinLength,
  IsOptional,
} from 'class-validator';
import { UserRole } from '../models/user.model';

export class CreateUserDto {
  @IsString({ message: 'שם פרטי חייב להיות מחרוזת' })
  @MinLength(2, { message: 'שם פרטי חייב להכיל לפחות 2 תווים' })
  firstName: string;

  @IsString({ message: 'שם משפחה חייב להיות מחרוזת' })
  @MinLength(2, { message: 'שם משפחה חייב להכיל לפחות 2 תווים' })
  lastName: string;

  @IsString({ message: 'שם משתמש חייב להיות מחרוזת' })
  @MinLength(3, { message: 'שם משתמש חייב להכיל לפחות 3 תווים' })
  username: string;

  @IsEmail({}, { message: 'אימייל לא תקין' })
  email: string;

  @IsString({ message: 'סיסמה חייבת להיות מחרוזת' })
  @MinLength(6, { message: 'סיסמה חייבת להכיל לפחות 6 תווים' })
  password: string;

  @IsOptional()
  @IsEnum(UserRole, { message: 'תפקיד לא תקין' })
  role?: UserRole;
}
```

### 3.4: יצירת Users Service
עדכן את הקובץ: `src/users/users.service.ts`

```typescript
import { Injectable, ConflictException } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { User, UserRole } from './models/user.model';
import { CreateUserDto } from './dto/create-user.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(
    @InjectModel(User)
    private userModel: typeof User,
  ) {}

  // יצירת משתמש חדש
  async create(createUserDto: CreateUserDto): Promise<User> {
    // בדיקה אם המשתמש כבר קיים
    const existingUser = await this.userModel.findOne({
      where: {
        [Op.or]: [
          { username: createUserDto.username },
          { email: createUserDto.email },
        ],
      },
    });

    if (existingUser) {
      throw new ConflictException('שם משתמש או אימייל כבר קיימים במערכת');
    }

    // הצפנת הסיסמה
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);

    // יצירת המשתמש
    const user = await this.userModel.create({
      ...createUserDto,
      password: hashedPassword,
    });

    return user;
  }

  // חיפוש משתמש לפי שם משתמש
  async findByUsername(username: string): Promise<User | null> {
    return this.userModel.findOne({ where: { username } });
  }

  // חיפוש משתמש לפי ID
  async findById(id: number): Promise<User | null> {
    return this.userModel.findByPk(id);
  }

  // חיפוש משתמש לפי אימייל
  async findByEmail(email: string): Promise<User | null> {
    return this.userModel.findOne({ where: { email } });
  }

  // קבלת כל המשתמשים
  async findAll(): Promise<User[]> {
    return this.userModel.findAll({
      attributes: { exclude: ['password'] }, // לא מחזירים את הסיסמה
    });
  }
}
```

### 3.5: עדכון Users Module
עדכן את הקובץ: `src/users/users.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './models/user.model';

@Module({
  imports: [SequelizeModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // חשוב! נייצא את הסרביס לשימוש במודולים אחרים
})
export class UsersModule {}
```

---

## 🔐 **שלב 4: יצירת מודול ה-Authentication**

### 4.1: יצירת המבנה
```bash
nest g module auth
nest g controller auth
nest g service auth
```

### 4.2: יצירת DTOs לאימות
צור קובץ: `src/auth/dto/login.dto.ts`

```typescript
import { IsString, MinLength } from 'class-validator';

export class LoginDto {
  @IsString({ message: 'שם משתמש חייב להיות מחרוזת' })
  username: string;

  @IsString({ message: 'סיסמה חייבת להיות מחרוזת' })
  @MinLength(6, { message: 'סיסמה חייבת להכיל לפחות 6 תווים' })
  password: string;
}
```

צור קובץ: `src/auth/dto/register.dto.ts`

```typescript
import { CreateUserDto } from '../../users/dto/create-user.dto';

// נשתמש באותו DTO של יצירת משתמש
export class RegisterDto extends CreateUserDto {}
```

### 4.3: יצירת Auth Service
עדכן את הקובץ: `src/auth/auth.service.ts`

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { LoginDto } from './dto/login.dto';
import { RegisterDto } from './dto/register.dto';
import * as bcrypt from 'bcrypt';
import { User } from '../users/models/user.model';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  // רישום משתמש חדש
  async register(registerDto: RegisterDto) {
    const user = await this.usersService.create(registerDto);
    
    // יצירת JWT token
    const token = this.generateToken(user);
    
    return {
      message: 'משתמש נרשם בהצלחה',
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
        role: user.role,
      },
      access_token: token,
    };
  }

  // התחברות משתמש
  async login(loginDto: LoginDto) {
    // חיפוש המשתמש
    const user = await this.usersService.findByUsername(loginDto.username);
    
    if (!user) {
      throw new UnauthorizedException('שם משתמש או סיסמה שגויים');
    }

    // בדיקת הסיסמה
    const isPasswordValid = await bcrypt.compare(
      loginDto.password,
      user.password,
    );

    if (!isPasswordValid) {
      throw new UnauthorizedException('שם משתמש או סיסמה שגויים');
    }

    // בדיקה אם המשתמש פעיל
    if (!user.isActive) {
      throw new UnauthorizedException('חשבון המשתמש אינו פעיל');
    }

    // יצירת JWT token
    const token = this.generateToken(user);

    return {
      message: 'התחברות הצליחה',
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
        role: user.role,
      },
      access_token: token,
    };
  }

  // יצירת JWT token
  private generateToken(user: User): string {
    const payload = {
      sub: user.id, // subject = מזהה המשתמש
      username: user.username,
      email: user.email,
      role: user.role,
    };

    return this.jwtService.sign(payload);
  }

  // בדיקת תקינות token והחזרת המשתמש
  async validateToken(payload: any): Promise<User> {
    const user = await this.usersService.findById(payload.sub);
    
    if (!user || !user.isActive) {
      throw new UnauthorizedException('משתמש לא תקין');
    }

    return user;
  }
}
```

### 4.4: יצירת JWT Strategy
צור קובץ: `src/auth/strategies/jwt.strategy.ts`

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { AuthService } from '../auth.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({
      // מאיפה לחלץ את ה-token
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      // האם לאפשר tokens שפגו
      ignoreExpiration: false,
      // המפתח הסודי (צריך להיות באותו מפתח שאיתו חתמנו על ה-token)
      secretOrKey: process.env.JWT_SECRET || 'your-secret-key-change-this',
    });
  }

  // פונקציה שמופעלת כשה-token תקין
  async validate(payload: any) {
    const user = await this.authService.validateToken(payload);
    
    if (!user) {
      throw new UnauthorizedException();
    }

    // מה שנחזיר כאן יישמר ב-request.user
    return {
      id: user.id,
      username: user.username,
      email: user.email,
      role: user.role,
    };
  }
}
```

**הסבר על JWT Strategy**:
- `jwtFromRequest`: אומר ל-Passport איפה למצוא את ה-token (בכותרת Authorization)
- `secretOrKey`: המפתח הסודי לבדיקת התקינות של ה-token
- `validate`: פונקציה שרצה אחרי שה-token אומת, ומחזירה את פרטי המשתמש

### 4.5: יצירת Auth Controller
עדכן את הקובץ: `src/auth/auth.controller.ts`

```typescript
import { Controller, Post, Body, HttpCode, HttpStatus } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LoginDto } from './dto/login.dto';
import { RegisterDto } from './dto/register.dto';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  // רישום משתמש חדש
  @Post('register')
  async register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  // התחברות
  @Post('login')
  @HttpCode(HttpStatus.OK)
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

### 4.6: עדכון Auth Module
עדכן את הקובץ: `src/auth/auth.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    UsersModule, // ייבוא מודול המשתמשים
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'your-secret-key-change-this',
      signOptions: {
        expiresIn: '24h', // תוקף ה-token
      },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService, JwtModule], // נייצא לשימוש במודולים אחרים
})
export class AuthModule {}
```

---

## 🛡️ **שלב 5: יצירת Guards**

### 5.1: הבנת Guards
**Guard** = שומר
- בודק אם המשתמש יכול לגשת ל-endpoint מסוים
- רץ לפני ה-handler של ה-Controller
- יכול לאשר או לחסום גישה

### 5.2: יצירת JWT Auth Guard
צור קובץ: `src/auth/guards/jwt-auth.guard.ts`

```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // בדיקה אם ה-endpoint מסומן כ-Public
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true; // מאפשרים גישה ללא אימות
    }

    // אחרת, מריצים את בדיקת ה-JWT הרגילה
    return super.canActivate(context);
  }
}
```

### 5.3: יצירת Roles Guard
צור קובץ: `src/auth/guards/roles.guard.ts`

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { UserRole } from '../../users/models/user.model';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // קבלת התפקידים הנדרשים שהוגדרו ב-decorator
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>(
      'roles',
      [context.getHandler(), context.getClass()],
    );

    // אם לא הוגדרו תפקידים נדרשים, מאפשרים גישה
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    // קבלת המשתמש מה-request
    const request = context.switchToHttp().getRequest();
    const user = request.user;

    // בדיקה אם המשתמש קיים ויש לו אחד מהתפקידים הנדרשים
    return user && requiredRoles.includes(user.role);
  }
}
```

**הסבר על הקוד**:
1. `Reflector` - מאפשר לקרוא metadata שהוגדרו על ה-handlers
2. `requiredRoles` - התפקידים שהוגדרו ב-decorator `@Roles()`
3. `request.user` - המשתמש ששמרנו ב-JWT Strategy
4. בודקים אם למשתמש יש אחד מהתפקידים הנדרשים

### 5.4: יצירת Permissions Guard
צור קובץ: `src/auth/guards/permissions.guard.ts`

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

// הגדרת enum להרשאות
export enum Permission {
  // הרשאות משתמשים
  CREATE_USER = 'create:user',
  READ_USER = 'read:user',
  UPDATE_USER = 'update:user',
  DELETE_USER = 'delete:user',
  
  // הרשאות תלמידים
  CREATE_STUDENT = 'create:student',
  READ_STUDENT = 'read:student',
  UPDATE_STUDENT = 'update:student',
  DELETE_STUDENT = 'delete:student',
  
  // הרשאות קורסים
  CREATE_COURSE = 'create:course',
  READ_COURSE = 'read:course',
  UPDATE_COURSE = 'update:course',
  DELETE_COURSE = 'delete:course',
  
  // הרשאות ציונים
  CREATE_GRADE = 'create:grade',
  READ_GRADE = 'read:grade',
  UPDATE_GRADE = 'update:grade',
  DELETE_GRADE = 'delete:grade',
}

// מפת הרשאות לפי תפקיד
export const RolePermissions = {
  admin: [
    // מנהל יכול הכל
    ...Object.values(Permission),
  ],
  teacher: [
    // מורה יכול לקרוא משתמשים
    Permission.READ_USER,
    // מורה יכול הכל על תלמידים
    Permission.CREATE_STUDENT,
    Permission.READ_STUDENT,
    Permission.UPDATE_STUDENT,
    // מורה יכול לקרוא ולעדכן קורסים
    Permission.READ_COURSE,
    Permission.UPDATE_COURSE,
    // מורה יכול הכל על ציונים
    Permission.CREATE_GRADE,
    Permission.READ_GRADE,
    Permission.UPDATE_GRADE,
    Permission.DELETE_GRADE,
  ],
  student: [
    // תלמיד יכול לקרוא את עצמו
    Permission.READ_USER,
    // תלמיד יכול לקרוא תלמידים אחרים
    Permission.READ_STUDENT,
    // תלמיד יכול לקרוא קורסים
    Permission.READ_COURSE,
    // תלמיד יכול לקרוא את הציונים שלו
    Permission.READ_GRADE,
  ],
};

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // קבלת ההרשאות הנדרשות
    const requiredPermissions = this.reflector.getAllAndOverride<Permission[]>(
      'permissions',
      [context.getHandler(), context.getClass()],
    );

    // אם לא הוגדרו הרשאות, מאפשרים גישה
    if (!requiredPermissions || requiredPermissions.length === 0) {
      return true;
    }

    // קבלת המשתמש
    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      return false;
    }

    // קבלת ההרשאות של התפקיד של המשתמש
    const userPermissions = RolePermissions[user.role] || [];

    // בדיקה אם למשתמש יש את כל ההרשאות הנדרשות
    return requiredPermissions.every((permission) =>
      userPermissions.includes(permission),
    );
  }
}
```

---

## 🎨 **שלב 6: יצירת Decorators**

### 6.1: Public Decorator
צור קובץ: `src/auth/decorators/public.decorator.ts`

```typescript
import { SetMetadata } from '@nestjs/common';

// decorator שמסמן endpoint כפומבי (לא דורש אימות)
export const Public = () => SetMetadata('isPublic', true);
```

**שימוש**:
```typescript
@Public()
@Get('public-data')
getPublicData() {
  return { message: 'זה endpoint פומבי' };
}
```

### 6.2: Roles Decorator
צור קובץ: `src/auth/decorators/roles.decorator.ts`

```typescript
import { SetMetadata } from '@nestjs/common';
import { UserRole } from '../../users/models/user.model';

// decorator להגדרת תפקידים נדרשים
export const Roles = (...roles: UserRole[]) => SetMetadata('roles', roles);
```

**שימוש**:
```typescript
@Roles(UserRole.ADMIN, UserRole.TEACHER)
@Get('teachers-only')
getTeachersData() {
  return { message: 'רק מורים ומנהלים יכולים לראות את זה' };
}
```

### 6.3: Permissions Decorator
צור קובץ: `src/auth/decorators/permissions.decorator.ts`

```typescript
import { SetMetadata } from '@nestjs/common';
import { Permission } from '../guards/permissions.guard';

// decorator להגדרת הרשאות נדרשות
export const RequirePermissions = (...permissions: Permission[]) =>
  SetMetadata('permissions', permissions);
```

**שימוש**:
```typescript
@RequirePermissions(Permission.CREATE_STUDENT, Permission.UPDATE_STUDENT)
@Post('students')
createStudent() {
  return { message: 'רק מי שיש לו הרשאות CREATE_STUDENT יכול ליצור תלמיד' };
}
```

### 6.4: Current User Decorator
צור קובץ: `src/auth/decorators/current-user.decorator.ts`

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

// decorator לקבלת המשתמש הנוכחי
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

**שימוש**:
```typescript
@Get('profile')
getProfile(@CurrentUser() user: any) {
  return {
    message: `שלום ${user.username}`,
    user: user,
  };
}
```

---

## 🔧 **שלב 7: הפעלת ה-Guards באפליקציה**

### 7.1: הפעלה גלובלית
עדכן את הקובץ: `src/main.ts`

```typescript
import { NestFactory, Reflector } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // הפעלת Validation Pipe גלובלי
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // מסיר שדות שלא מוגדרים ב-DTO
      forbidNonWhitelisted: true, // זורק שגיאה על שדות לא מוגדרים
      transform: true, // המרה אוטומטית לטיפוסים
    }),
  );

  // הפעלת JWT Guard גלובלי
  const reflector = app.get(Reflector);
  app.useGlobalGuards(new JwtAuthGuard(reflector));

  await app.listen(3000);
  console.log('🚀 Server is running on http://localhost:3000');
}
bootstrap();
```

**הסבר**:
- עכשיו כל ה-endpoints מוגנים באופן אוטומטי
- צריך להשתמש ב-`@Public()` כדי לאפשר גישה ללא אימות

### 7.2: הפעלה ספציפית ב-Controller
אם לא רוצים גלובלי, אפשר להשתמש כך:

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';
import { RolesGuard } from './auth/guards/roles.guard';
import { Roles } from './auth/decorators/roles.decorator';
import { UserRole } from './users/models/user.model';

@Controller('students')
@UseGuards(JwtAuthGuard, RolesGuard) // הגנה על כל ה-Controller
export class StudentsController {
  @Get()
  @Roles(UserRole.ADMIN, UserRole.TEACHER) // רק מנהל ומורה
  findAll() {
    return { message: 'רשימת תלמידים' };
  }

  @Get('my-data')
  @Roles(UserRole.STUDENT) // רק תלמידים
  getMyData(@CurrentUser() user: any) {
    return { message: `הנתונים של ${user.username}` };
  }
}
```

---

## 📝 **שלב 8: דוגמאות שימוש מלאות**

### 8.1: Controller עם Roles
צור קובץ: `src/students/students.controller.ts`

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
} from '@nestjs/common';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { Roles } from '../auth/decorators/roles.decorator';
import { CurrentUser } from '../auth/decorators/current-user.decorator';
import { UserRole } from '../users/models/user.model';
import { StudentsService } from './students.service';
import { CreateStudentDto } from './dto/create-student.dto';

@Controller('students')
@UseGuards(JwtAuthGuard, RolesGuard)
export class StudentsController {
  constructor(private studentsService: StudentsService) {}

  // רק מנהל ומורה יכולים לראות את כל התלמידים
  @Get()
  @Roles(UserRole.ADMIN, UserRole.TEACHER)
  async findAll() {
    return this.studentsService.findAll();
  }

  // תלמיד יכול לראות רק את עצמו
  @Get('me')
  @Roles(UserRole.STUDENT)
  async getMyProfile(@CurrentUser() user: any) {
    return this.studentsService.findByUserId(user.id);
  }

  // רק מנהל יכול ליצור תלמיד חדש
  @Post()
  @Roles(UserRole.ADMIN)
  async create(@Body() createStudentDto: CreateStudentDto) {
    return this.studentsService.create(createStudentDto);
  }

  // רק מנהל יכול למחוק תלמיד
  @Delete(':id')
  @Roles(UserRole.ADMIN)
  async remove(@Param('id') id: number) {
    return this.studentsService.remove(id);
  }
}
```

### 8.2: Controller עם Permissions
צור קובץ: `src/courses/courses.controller.ts`

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
} from '@nestjs/common';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { PermissionsGuard } from '../auth/guards/permissions.guard';
import { RequirePermissions } from '../auth/decorators/permissions.decorator';
import { Permission } from '../auth/guards/permissions.guard';
import { CurrentUser } from '../auth/decorators/current-user.decorator';
import { CoursesService } from './courses.service';
import { CreateCourseDto } from './dto/create-course.dto';

@Controller('courses')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class CoursesController {
  constructor(private coursesService: CoursesService) {}

  // כולם יכולים לקרוא קורסים
  @Get()
  @RequirePermissions(Permission.READ_COURSE)
  async findAll() {
    return this.coursesService.findAll();
  }

  // רק מי שיש לו הרשאה ליצור קורס
  @Post()
  @RequirePermissions(Permission.CREATE_COURSE)
  async create(@Body() createCourseDto: CreateCourseDto) {
    return this.coursesService.create(createCourseDto);
  }

  // רק מי שיש לו הרשאה לעדכן קורס
  @Put(':id')
  @RequirePermissions(Permission.UPDATE_COURSE)
  async update(
    @Param('id') id: number,
    @Body() updateCourseDto: any,
  ) {
    return this.coursesService.update(id, updateCourseDto);
  }

  // רק מי שיש לו הרשאה למחוק קורס
  @Delete(':id')
  @RequirePermissions(Permission.DELETE_COURSE)
  async remove(@Param('id') id: number) {
    return this.coursesService.remove(id);
  }
}
```

### 8.3: Controller משולב (Roles + Permissions)
```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  UseGuards,
} from '@nestjs/common';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { PermissionsGuard } from '../auth/guards/permissions.guard';
import { Roles } from '../auth/decorators/roles.decorator';
import { RequirePermissions } from '../auth/decorators/permissions.decorator';
import { UserRole } from '../users/models/user.model';
import { Permission } from '../auth/guards/permissions.guard';
import { CurrentUser } from '../auth/decorators/current-user.decorator';
import { GradesService } from './grades.service';

@Controller('grades')
@UseGuards(JwtAuthGuard, RolesGuard, PermissionsGuard)
export class GradesController {
  constructor(private gradesService: GradesService) {}

  // תלמיד יכול לראות רק את הציונים שלו
  @Get('my-grades')
  @Roles(UserRole.STUDENT)
  async getMyGrades(@CurrentUser() user: any) {
    return this.gradesService.findByStudentId(user.id);
  }

  // מורה יכול לראות ציונים של כל התלמידים
  @Get('all')
  @Roles(UserRole.TEACHER, UserRole.ADMIN)
  @RequirePermissions(Permission.READ_GRADE)
  async getAllGrades() {
    return this.gradesService.findAll();
  }

  // רק מורה ומנהל יכולים להוסיף ציון
  @Post()
  @Roles(UserRole.TEACHER, UserRole.ADMIN)
  @RequirePermissions(Permission.CREATE_GRADE)
  async createGrade(@Body() createGradeDto: any) {
    return this.gradesService.create(createGradeDto);
  }
}
```

---

## 🧪 **שלב 9: בדיקת המערכת**

### 9.1: בדיקה עם Postman או Thunder Client

#### צעד 1: רישום משתמש חדש
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "firstName": "יוסי",
  "lastName": "כהן",
  "username": "yossi",
  "email": "yossi@example.com",
  "password": "123456",
  "role": "student"
}
```

**תגובה צפויה**:
```json
{
  "message": "משתמש נרשם בהצלחה",
  "user": {
    "id": 1,
    "username": "yossi",
    "email": "yossi@example.com",
    "role": "student"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### צעד 2: התחברות
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "username": "yossi",
  "password": "123456"
}
```

#### צעד 3: שליחת בקשה מוגנת
```http
GET http://localhost:3000/students
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**תגובה אם אין הרשאה**:
```json
{
  "statusCode": 403,
  "message": "Forbidden resource"
}
```

### 9.2: קוד לבדיקה מהירה
צור קובץ: `test-auth.http` (אם יש לך extension REST Client ב-VS Code)

```http
### רישום כמנהל
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "firstName": "אדמין",
  "lastName": "ראשי",
  "username": "admin",
  "email": "admin@example.com",
  "password": "admin123",
  "role": "admin"
}

### התחברות כמנהל
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "admin123"
}

### שמירת ה-Token (החלף את TOKEN כאן)
@token = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

### בדיקת endpoint מוגן
GET http://localhost:3000/students
Authorization: Bearer {{token}}

### רישום כתלמיד
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "firstName": "דני",
  "lastName": "לוי",
  "username": "danny",
  "email": "danny@example.com",
  "password": "123456",
  "role": "student"
}
```

---

## 🎯 **שלב 10: סיכום ותרגילים**

### סיכום מה למדנו:
1. ✅ יצירת מודל User עם תפקידים
2. ✅ הצפנת סיסמאות עם bcrypt
3. ✅ יצירת מערכת Authentication עם JWT
4. ✅ יצירת Guards לבדיקת אימות והרשאות
5. ✅ יצירת Decorators לשימוש נוח
6. ✅ מערכת Roles (תפקידים)
7. ✅ מערכת Permissions (הרשאות מפורטות)
8. ✅ שילוב והפעלה במערכת

### תרגילים לתרגול:

#### תרגיל 1: הוספת תפקיד חדש
הוסף תפקיד `PARENT` (הורה) שיכול:
- לראות את הציונים של הילדים שלו
- לראות את הנוכחות של הילדים שלו
- לא יכול לערוך שום דבר

#### תרגיל 2: הרשאות מותנות
צור Guard שבודק:
- תלמיד יכול לראות רק את הציונים שלו
- מורה יכול לראות ציונים רק של הקורסים שהוא מלמד
- מנהל יכול לראות הכל

#### תרגיל 3: Permissions מתקדם
צור מערכת הרשאות דינמית שנשמרת במסד הנתונים:
- טבלת `permissions`
- טבלת ביניים `role_permissions`
- טעינת הרשאות מהמסד נתונים במקום מה-enum

#### תרגיל 4: Refresh Tokens
הוסף מערכת Refresh Tokens:
- Access Token לשעה אחת
- Refresh Token לשבוע
- Endpoint לרענון ה-Access Token

---

## ❓ שאלות נפוצות

### 1. מה ההבדל בין Roles ל-Permissions?
**Roles** = תפקיד כללי (תלמיד, מורה, מנהל)
**Permissions** = הרשאות ספציפיות (יצירה, קריאה, עדכון, מחיקה)

### 2. מתי להשתמש ב-Roles ומתי ב-Permissions?
- **Roles**: כשיש תפקידים ברורים במערכת
- **Permissions**: כשצריך שליטה מדויקת על פעולות ספציפיות
- **שניהם**: במערכות מורכבות

### 3. איך שומרים את ה-Token בצד הלקוח?
אפשרויות:
1. **LocalStorage** - פשוט אבל פחות בטוח
2. **SessionStorage** - נמחק כשסוגרים את הדפדפן
3. **Cookie (HttpOnly)** - הכי בטוח

### 4. מה לעשות אם ה-Token נגנב?
- הגדר `expiresIn` קצר (שעה-שעתיים)
- השתמש ב-Refresh Tokens
- אפשר ביטול Tokens (blacklist)
- הוסף רישום IP ו-Device

### 5. איך בודקים את ההרשאות בצד הלקוח?
אפשר לפענח את ה-JWT בצד הלקוח ולקרוא את ה-role:
```javascript
// בJavaScript
function parseJwt(token) {
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

const token = localStorage.getItem('access_token');
const payload = parseJwt(token);
console.log(payload.role); // admin / teacher / student
```

---

## 🔒 **שלב 11: אבטחה מתקדמת (Bonus)**

### 11.1: Rate Limiting לפי תפקיד
```typescript
import { Injectable } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';

@Injectable()
export class RoleBasedThrottlerGuard extends ThrottlerGuard {
  protected async getTracker(req: Record<string, any>): Promise<string> {
    const user = req.user;
    
    // מנהל - ללא הגבלה
    if (user?.role === 'admin') {
      return `admin-${user.id}`;
    }
    
    // משתמשים רגילים - 100 בקשות לדקה
    return req.ip;
  }

  protected async getLimit(context: ExecutionContext): Promise<number> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    if (user?.role === 'admin') {
      return 1000; // מנהל - 1000 בקשות
    }
    
    return 100; // משתמש רגיל - 100 בקשות
  }
}
```

### 11.2: Audit Log
צור מערכת לרישום כל הפעולות:
```typescript
// audit-log.model.ts
@Table({ tableName: 'audit_logs' })
export class AuditLog extends Model {
  @Column
  userId: number;

  @Column
  action: string; // CREATE, UPDATE, DELETE, READ

  @Column
  resource: string; // students, courses, grades

  @Column
  resourceId: number;

  @Column(DataType.JSON)
  changes: any;

  @Column
  ipAddress: string;

  @Column
  userAgent: string;
}

// audit.interceptor.ts
@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(
    @InjectModel(AuditLog) private auditLogModel: typeof AuditLog,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { user, method, url, body, ip, headers } = request;

    return next.handle().pipe(
      tap(async (response) => {
        // רישום הפעולה
        await this.auditLogModel.create({
          userId: user?.id,
          action: method,
          resource: url,
          changes: body,
          ipAddress: ip,
          userAgent: headers['user-agent'],
        });
      }),
    );
  }
}
```

---

## 📚 משאבים נוספים

### מסמכים רשמיים:
- [NestJS Authentication](https://docs.nestjs.com/security/authentication)
- [NestJS Authorization](https://docs.nestjs.com/security/authorization)
- [Passport JWT](http://www.passportjs.org/packages/passport-jwt/)

### מאמרים מומלצים:
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- OAuth2 & OpenID Connect

---

**זהו! סיימנו מדריך מקיף על Authorization ו-Permissions! 🎉**

### מה הלאה?
1. תרגלו את כל השלבים
2. נסו ליצור מערכת משלכם
3. הוסיפו פיצ'רים נוספים
4. קראו על CASL ו-Casbin למערכות הרשאות מתקדמות יותר

**בהצלחה! 💪**

---

## 📊 שלב 7: Queries מתקדמים

### תרגיל 7.1: Filtering ו-Sorting
**מטרה**: להוסיף אפשרויות סינון ומיון

**משימות**:
1. הוסף query parameters ל-endpoint של קבלת תלמידים:
   - `age` - סינון לפי גיל
   - `course` - סינון לפי קורס
   - `sortBy` - מיון לפי שדה (name, age, createdAt)
   - `order` - סדר מיון (ASC/DESC)
2. מימוש הלוגיקה ב-Service
3. בדוק עם queries שונים

**דרישות**:
- השתמש ב-`@Query` decorator
- השתמש ב-Sequelize operators

---

### תרגיל 7.2: Pagination
**מטרה**: להוסיף pagination לתוצאות

**משימות**:
1. הוסף parameters:
   - `page` - מספר עמוד (ברירת מחדל: 1)
   - `limit` - כמות תוצאות לעמוד (ברירת מחדל: 10)
2. החזר מידע נוסף:
   - סך הכל תוצאות
   - מספר עמודים
   - עמוד נוכחי
3. מימוש עם Sequelize

**דרישות**:
- השתמש ב-`offset` ו-`limit`
- החזר מבנה תגובה עקבי

---

### תרגיל 7.3: Search
**מטרה**: להוסיף חיפוש טקסטואלי

**משימות**:
1. הוסף parameter `search` שמחפש בשמות ובאימיילים
2. הוסף חיפוש לקורסים
3. צור endpoint לחיפוש כללי במערכת
4. הוסף debouncing בצד הלקוח (רעיון)

**דרישות**:
- השתמש ב-`Op.like` או `Op.iLike`
- תמיכה בחיפוש חלקי

---

## 🧪 שלב 8: Testing

### תרגיל 8.1: Unit Tests
**מטרה**: לכתוב בדיקות יחידה

**משימות**:
1. כתוב unit tests ל-`StudentsService`:
   - בדיקת `findAll`
   - בדיקת `findOne` עם ID קיים
   - בדיקת `findOne` עם ID לא קיים
   - בדיקת `create` עם נתונים תקינים
   - בדיקת `create` עם נתונים לא תקינים
2. השתמש ב-mocks למסד הנתונים
3. הרץ את הבדיקות ווודא שעוברות

**דרישות**:
- כיסוי של לפחות 80%
- השתמש ב-Jest

---

### תרגיל 8.2: Integration Tests
**מטרה**: לבדוק את האינטגרציה בין הרכיבים

**משימות**:
1. כתוב integration tests ל-`StudentsController`:
   - בדיקת GET /students
   - בדיקת GET /students/:id
   - בדיקת POST /students
   - בדיקת PUT /students/:id
   - בדיקת DELETE /students/:id
2. השתמש במסד נתונים ייעודי לבדיקות
3. נקה את מסד הנתונים לפני כל בדיקה

**דרישות**:
- בדוק גם status codes וגם תוכן התגובה
- בדוק edge cases

---

### תרגיל 8.3: E2E Tests
**מטרה**: לבדוק את כל המערכת מקצה לקצה

**משימות**:
1. כתוב E2E tests שבודקים תהליכים שלמים:
   - רישום משתמש -> התחברות -> יצירת תלמיד -> עדכון -> מחיקה
   - רישום לקורס -> צפייה בקורסים -> ביטול רישום
2. בדוק authentication ו-authorization
3. בדוק error handling

**דרישות**:
- השתמש ב-`supertest`
- נקה את מסד הנתונים אחרי כל בדיקה

---

## 🚀 שלב 9: Advanced Features

### תרגיל 9.1: Logging
**מטרה**: להוסיף מערכת logging

**משימות**:
1. התקן את `winston` או השתמש ב-Logger של NestJS
2. הוסף logging למקומות הבאים:
   - כל בקשה שנכנסת (middleware)
   - כל שגיאה
   - פעולות CRUD
3. שמור logs לקובץ
4. צור רמות logging שונות (info, warn, error)

**דרישות**:
- לוגים צריכים להכיל timestamp, level, message, context
- לוגים שגיאות צריכים להכיל stack trace

---

### תרגיל 9.2: Caching
**מטרה**: להוסיף caching לשיפור ביצועים

**משימות**:
1. התקן את `@nestjs/cache-manager`
2. הגדר Redis כ-cache store
3. הוסף caching ל:
   - רשימת תלמידים
   - פרטי קורס
4. הגדר TTL (Time To Live) מתאים
5. אפס cache כשיש עדכון

**דרישות**:
- השתמש ב-`@CacheInterceptor`
- בדוק שה-caching עובד

---

### תרגיל 9.3: Rate Limiting
**מטרה**: להגביל קצב בקשות

**משימות**:
1. התקן את `@nestjs/throttler`
2. הגדר מגבלות:
   - 100 בקשות לדקה למשתמש רגיל
   - 1000 בקשות לדקה למשתמש מורשה
3. החזר תגובות מתאימות כשחורגים מהמגבלה
4. בדוק עם Postman או כלי דומה

**דרישות**:
- החזר 429 (Too Many Requests) כשחורגים
- הוסף headers עם מידע על המגבלות

---

## 📈 שלב 10: Monitoring ו-Production

### תרגיל 10.1: Health Checks
**מטרה**: להוסיף בדיקות בריאות למערכת

**משימות**:
1. התקן את `@nestjs/terminus`
2. צור endpoint `/health` שבודק:
   - חיבור למסד נתונים
   - זיכרון פנוי
   - שטח דיסק
3. הוסף health check ל-Docker Compose
4. בדוק שזה עובד

**דרישות**:
- החזר status 200 אם הכל תקין
- החזר status 503 אם משהו לא תקין

---

### תרגיל 10.2: Environment Configuration
**מטרה**: לנהל הגדרות לפי סביבה

**משימות**:
1. התקן את `@nestjs/config`
2. צור קבצי `.env` שונים:
   - `.env.development`
   - `.env.test`
   - `.env.production`
3. טען את ההגדרות המתאימות לכל סביבה
4. השתמש ב-`ConfigService` בקוד

**דרישות**:
- אל תעלה את קבצי `.env` ל-Git
- וודא ש-`.env.example` קיים

---

### תרגיל 10.3: Deployment
**מטרה**: להעלות את האפליקציה לענן

**משימות**:
1. בחר פלטפורמה (Heroku, AWS, GCP, DigitalOcean)
2. הכן את האפליקציה ל-production:
   - הגדר `NODE_ENV=production`
   - כבה את `synchronize` ב-Sequelize
   - הגדר CORS נכון
3. העלה את האפליקציה
4. בדוק שהיא עובדת

**דרישות**:
- השתמש במשתני סביבה בצורה בטוחה
- הגדר HTTPS
- הפעל compression

---

## 🎓 פרויקט גמר: מערכת ניהול בית ספר מלאה

### דרישות הפרויקט
צור מערכת ניהול בית ספר מקיפה שכוללת:

#### ישויות (Entities):
1. **Students** - תלמידים
2. **Teachers** - מורים
3. **Courses** - קורסים
4. **Classes** - כיתות
5. **Assignments** - מ과제ים
6. **Grades** - ציונים
7. **Attendance** - נוכחות
8. **Announcements** - הודעות

#### פונקציונליות:
1. **אימות והרשאות**:
   - רישום והתחברות
   - תפקידים: תלמיד, מורה, מנהל
   - הרשאות שונות לכל תפקיד

2. **ניהול קורסים**:
   - יצירת קורסים
   - הקצאת מורים לקורסים
   - רישום תלמידים לקורסים
   - צפייה בתלמידים בקורס

3. **ניהול משימות**:
   - מורה יכול ליצור משימות לקורס
   - תלמיד יכול לצפות במשימות שלו
   - העלאת משימות (file upload)
   - מתן ציונים למשימות

4. **ניהול נוכחות**:
   - מורה רושם נוכחות לכל שיעור
   - תלמיד יכול לראות את הנוכחות שלו
   - דוח נוכחות חודשי

5. **ניהול ציונים**:
   - מורה נותן ציונים
   - חישוב ממוצע
   - דוח ציונים

6. **מערכת הודעות**:
   - מנהל יכול לפרסם הודעות
   - הודעות לפי תפקיד או קורס
   - התראות

#### דרישות טכניות:
- NestJS עם TypeScript
- Sequelize עם PostgreSQL
- JWT Authentication
- Role-Based Access Control
- Docker Compose לסביבת הפיתוח
- Validation עם class-validator
- Error handling מקיף
- Logging
- Unit tests ו-Integration tests
- API documentation עם Swagger
- Pagination, Filtering, Sorting לכל הרשימות
- Rate limiting
- File upload לmשימות

#### Bonus:
- WebSockets להתראות בזמן אמת
- Email notifications
- Redis caching
- Migrations עם Sequelize
- CI/CD pipeline
- Deployment לענן

---

## 📝 הערות חשובות

### טיפים ללמידה:
1. **התקדמו לאט** - אל תדלגו על שלבים
2. **נסו בעצמכם** - לפני לחפש בגוגל
3. **קראו תיעוד** - NestJS ו-Sequelize מתועדים מצוין
4. **תרגלו הרבה** - כתבו קוד כל יום
5. **טעו ולמדו** - שגיאות הן חלק מהלמידה

### משאבים מומלצים:
- [NestJS Documentation](https://docs.nestjs.com)
- [Sequelize Documentation](https://sequelize.org/docs/)
- [Docker Documentation](https://docs.docker.com)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com)

### איך לגשת לתרגילים:
1. קראו את המשימה כולה
2. תכננו איך תממשו
3. כתבו קוד
4. בדקו שהכל עובד
5. רפקטור אם צריך
6. עברו לתרגיל הבא

---

**בהצלחה! 🚀**
