# תרגילים ב-Sequelize למתחילים

מדריך תרגול מעשי ל-Sequelize ORM עם דוגמאות והסברים מפורטים.

## תוכן עניינים

1. [בסיס - הגדרת מודלים](#תרגיל-1-הגדרת-מודל-בסיסי)
2. [שדות ווולידציות](#תרגיל-2-שדות-ווולידציות)
3. [פעולות CRUD בסיסיות](#תרגיל-3-crud-בסיסי)
4. [שאילתות מתקדמות](#תרגיל-4-שאילתות-מתקדמות)
5. [יחסים בין טבלאות](#תרגיל-5-יחסים-relationships)
6. [Hooks](#תרגיל-6-hooks)
7. [Transactions](#תרגיל-7-transactions)
8. [Scopes](#תרגיל-8-scopes)

---

## תרגיל 1: הגדרת מודל בסיסי

### מטרה
ללמוד איך ליצור מודל פשוט ב-Sequelize עם שדות בסיסיים.

### דוגמה: מודל Product (מוצר)

```typescript
import { Table, Column, Model, DataType } from 'sequelize-typescript';

@Table({
  tableName: 'products',
  timestamps: true,
})
export class Product extends Model {
  @Column({
    type: DataType.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  })
  id: number;

  @Column({
    type: DataType.STRING(100),
    allowNull: false,
  })
  name: string;

  @Column({
    type: DataType.DECIMAL(10, 2),
    allowNull: false,
  })
  price: number;

  @Column({
    type: DataType.INTEGER,
    defaultValue: 0,
  })
  stock: number;

  @Column({
    type: DataType.TEXT,
    allowNull: true,
  })
  description: string;
}
```

### הסבר
- `@Table` - מגדיר את הטבלה במסד הנתונים
- `timestamps: true` - מוסיף אוטומטית `createdAt` ו-`updatedAt`
- `@Column` - מגדיר עמודה בטבלה
- `DataType` - סוג הנתונים (STRING, INTEGER, DECIMAL, וכו')
- `allowNull` - האם מותר ערך NULL
- `defaultValue` - ערך ברירת מחדל

### תרגיל עצמאי 1.1
צור מודל `Book` עם השדות:
- `id` (מפתח ראשי)
- `title` (כותרת, חובה)
- `author` (מחבר, חובה)
- `publishYear` (שנת הוצאה)
- `isbn` (מספר ISBN, ייחודי)

<details>
<summary>פתרון</summary>

```typescript
import { Table, Column, Model, DataType } from 'sequelize-typescript';

@Table({
  tableName: 'books',
  timestamps: true,
})
export class Book extends Model {
  @Column({
    type: DataType.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  })
  id: number;

  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  title: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  author: string;

  @Column({
    type: DataType.INTEGER,
  })
  publishYear: number;

  @Column({
    type: DataType.STRING,
    unique: true,
  })
  isbn: string;
}
```
</details>

---

## תרגיל 2: שדות ווולידציות

### מטרה
ללמוד איך להוסיף וולידציות לשדות.

### דוגמה: מודל User עם וולידציות

```typescript
import { Table, Column, Model, DataType } from 'sequelize-typescript';

@Table({
  tableName: 'users',
  timestamps: true,
})
export class User extends Model {
  @Column({
    type: DataType.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  })
  id: number;

  @Column({
    type: DataType.STRING,
    allowNull: false,
    validate: {
      notEmpty: true,
      len: [2, 50], // אורך בין 2 ל-50 תווים
    },
  })
  firstName: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true, // בדיקת פורמט מייל
    },
  })
  email: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
    validate: {
      len: [6, 100], // לפחות 6 תווים
    },
  })
  password: string;

  @Column({
    type: DataType.INTEGER,
    validate: {
      min: 0, // גיל מינימלי
      max: 120, // גיל מקסימלי
    },
  })
  age: number;

  @Column({
    type: DataType.STRING,
    validate: {
      isUrl: true, // בדיקת פורמט URL
    },
  })
  website: string;

  @Column({
    type: DataType.ENUM('active', 'inactive', 'banned'),
    defaultValue: 'active',
  })
  status: string;
}
```

### וולידציות נפוצות ב-Sequelize:
- `notEmpty` - לא ריק
- `len: [min, max]` - אורך
- `isEmail` - פורמט אימייל
- `isUrl` - פורמט URL
- `isIP` - כתובת IP
- `isAlpha` - רק אותיות
- `isAlphanumeric` - אותיות ומספרים
- `isNumeric` - רק מספרים
- `min` / `max` - ערך מינימום/מקסימום
- `isIn: [[...]]` - ערכים מותרים

### תרגיל עצמאי 2.1
צור מודל `Employee` עם:
- `firstName` (2-30 תווים, חובה)
- `email` (פורמט אימייל, ייחודי)
- `phone` (פורמט טלפון)
- `salary` (מינימום 5000)
- `department` (רק אחד מ: 'IT', 'HR', 'Sales', 'Marketing')

<details>
<summary>פתרון</summary>

```typescript
@Table({
  tableName: 'employees',
  timestamps: true,
})
export class Employee extends Model {
  @Column({
    type: DataType.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  })
  id: number;

  @Column({
    type: DataType.STRING,
    allowNull: false,
    validate: {
      notEmpty: true,
      len: [2, 30],
    },
  })
  firstName: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true,
    },
  })
  email: string;

  @Column({
    type: DataType.STRING,
    validate: {
      is: /^[0-9]{10}$/, // 10 ספרות
    },
  })
  phone: string;

  @Column({
    type: DataType.DECIMAL(10, 2),
    validate: {
      min: 5000,
    },
  })
  salary: number;

  @Column({
    type: DataType.ENUM('IT', 'HR', 'Sales', 'Marketing'),
    allowNull: false,
  })
  department: string;
}
```
</details>

---

## תרגיל 3: CRUD בסיסי

### מטרה
ללמוד את ארבע הפעולות הבסיסיות.

### Create - יצירה

```typescript
// יצירת רשומה אחת
const user = await User.create({
  firstName: 'יוסי',
  email: 'yossi@example.com',
  password: 'password123',
  age: 25,
});

// יצירת מספר רשומות
const users = await User.bulkCreate([
  { firstName: 'דני', email: 'danny@example.com', password: 'pass123', age: 30 },
  { firstName: 'שרה', email: 'sara@example.com', password: 'pass456', age: 28 },
]);
```

### Read - קריאה

```typescript
// כל הרשומות
const allUsers = await User.findAll();

// לפי מפתח ראשי
const user = await User.findByPk(1);

// רשומה אחת לפי תנאי
const user = await User.findOne({
  where: { email: 'yossi@example.com' }
});

// ספירה
const count = await User.count();

// בדיקת קיום
const exists = await User.findOne({
  where: { email: 'test@example.com' }
}) !== null;
```

### Update - עדכון

```typescript
// עדכון דרך אובייקט
const user = await User.findByPk(1);
user.firstName = 'יוסף';
await user.save();

// עדכון ישיר
await User.update(
  { age: 26 },
  { where: { id: 1 } }
);

// עדכון מותנה
await User.update(
  { status: 'inactive' },
  { where: { age: { [Op.lt]: 18 } } }
);
```

### Delete - מחיקה

```typescript
// מחיקה דרך אובייקט
const user = await User.findByPk(1);
await user.destroy();

// מחיקה ישירה
await User.destroy({
  where: { id: 1 }
});

// מחיקת הכל (זהירות!)
await User.destroy({
  where: {},
  truncate: true
});
```

### תרגיל עצמאי 3.1
כתוב קוד שמבצע:
1. יוצר 3 מוצרים
2. מוצא מוצר לפי ID
3. מעדכן את המחיר שלו
4. מוחק מוצר עם stock = 0

<details>
<summary>פתרון</summary>

```typescript
// 1. יצירת 3 מוצרים
await Product.bulkCreate([
  { name: 'מחשב נייד', price: 3500, stock: 10 },
  { name: 'עכבר', price: 50, stock: 0 },
  { name: 'מקלדת', price: 150, stock: 5 },
]);

// 2. מציאת מוצר לפי ID
const product = await Product.findByPk(1);

// 3. עדכון מחיר
if (product) {
  product.price = 3200;
  await product.save();
}

// 4. מחיקת מוצרים עם stock = 0
await Product.destroy({
  where: { stock: 0 }
});
```
</details>

---

## תרגיל 4: שאילתות מתקדמות

### מטרה
ללמוד שאילתות מורכבות עם תנאים, מיונים ועוד.

### Operators (אופרטורים)

```typescript
import { Op } from 'sequelize';

// שווה ל
const users = await User.findAll({
  where: { age: 25 }
});

// גדול מ / קטן מ
const adults = await User.findAll({
  where: { age: { [Op.gte]: 18 } } // greater than or equal
});

const young = await User.findAll({
  where: { age: { [Op.lt]: 30 } } // less than
});

// בין
const middleAge = await User.findAll({
  where: { age: { [Op.between]: [25, 40] } }
});

// LIKE - חיפוש חלקי
const users = await User.findAll({
  where: { 
    firstName: { [Op.like]: '%יוסי%' } // מכיל "יוסי"
  }
});

// IN - ברשימה
const users = await User.findAll({
  where: { 
    status: { [Op.in]: ['active', 'pending'] }
  }
});

// NOT - שלילה
const users = await User.findAll({
  where: { 
    email: { [Op.not]: null }
  }
});

// OR - או
const users = await User.findAll({
  where: {
    [Op.or]: [
      { age: { [Op.gt]: 60 } },
      { status: 'vip' }
    ]
  }
});

// AND - וגם (ברירת מחדל)
const users = await User.findAll({
  where: {
    age: { [Op.gte]: 18 },
    status: 'active'
  }
});
```

### מיון, הגבלה, דילוג

```typescript
// מיון
const users = await User.findAll({
  order: [['age', 'DESC']] // מהגבוה לנמוך
});

// מיון לפי מספר שדות
const users = await User.findAll({
  order: [
    ['age', 'DESC'],
    ['firstName', 'ASC']
  ]
});

// הגבלת תוצאות
const users = await User.findAll({
  limit: 10
});

// דילוג (pagination)
const users = await User.findAll({
  limit: 10,
  offset: 20 // דף 3 (0-9, 10-19, 20-29)
});

// בחירת שדות ספציפיים
const users = await User.findAll({
  attributes: ['id', 'firstName', 'email']
});

// החרגת שדות
const users = await User.findAll({
  attributes: { exclude: ['password'] }
});
```

### Aggregation - צבירה

```typescript
// ספירה
const count = await User.count({
  where: { status: 'active' }
});

// סכום
const total = await Product.sum('price', {
  where: { stock: { [Op.gt]: 0 } }
});

// ממוצע
const avgAge = await User.aggregate('age', 'avg');

// מקסימום
const maxPrice = await Product.max('price');

// מינימום
const minPrice = await Product.min('price');

// קבוצות
const results = await Product.findAll({
  attributes: [
    'category',
    [sequelize.fn('COUNT', sequelize.col('id')), 'count'],
    [sequelize.fn('AVG', sequelize.col('price')), 'avgPrice']
  ],
  group: ['category']
});
```

### תרגיל עצמאי 4.1
כתוב שאילתות שמוצאות:
1. כל המשתמשים שהאימייל שלהם מכיל "gmail"
2. משתמשים בגיל 20-30, ממוין לפי שם
3. 5 המוצרים היקרים ביותר
4. ספירת משתמשים לפי status

<details>
<summary>פתרון</summary>

```typescript
// 1. מייל עם gmail
const gmailUsers = await User.findAll({
  where: {
    email: { [Op.like]: '%gmail%' }
  }
});

// 2. גילאים 20-30 ממוינים
const users = await User.findAll({
  where: {
    age: { [Op.between]: [20, 30] }
  },
  order: [['firstName', 'ASC']]
});

// 3. 5 מוצרים יקרים
const products = await Product.findAll({
  order: [['price', 'DESC']],
  limit: 5
});

// 4. ספירה לפי status
const statusCount = await User.findAll({
  attributes: [
    'status',
    [sequelize.fn('COUNT', sequelize.col('id')), 'count']
  ],
  group: ['status']
});
```
</details>

---

## תרגיל 5: יחסים (Relationships)

### מטרה
ללמוד איך ליצור קשרים בין טבלאות.

### One-to-One (אחד לאחד)

```typescript
// User יכול להיות לו Profile אחד
@Table({ tableName: 'users' })
export class User extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column
  firstName: string;

  @HasOne(() => Profile)
  profile: Profile;
}

@Table({ tableName: 'profiles' })
export class Profile extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @ForeignKey(() => User)
  @Column
  userId: number;

  @BelongsTo(() => User)
  user: User;

  @Column
  bio: string;

  @Column
  avatar: string;
}

// שימוש
const user = await User.findByPk(1, {
  include: [Profile]
});
console.log(user.profile.bio);
```

### One-to-Many (אחד לרבים)

```typescript
// User יכול לכתוב Posts רבים
@Table({ tableName: 'users' })
export class User extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column
  firstName: string;

  @HasMany(() => Post)
  posts: Post[];
}

@Table({ tableName: 'posts' })
export class Post extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @ForeignKey(() => User)
  @Column
  userId: number;

  @BelongsTo(() => User)
  user: User;

  @Column
  title: string;

  @Column(DataType.TEXT)
  content: string;
}

// שימוש
const user = await User.findByPk(1, {
  include: [Post]
});
console.log(user.posts.length); // כמה פוסטים יש למשתמש

// מציאת פוסט עם המשתמש שלו
const post = await Post.findByPk(1, {
  include: [User]
});
console.log(post.user.firstName);
```

### Many-to-Many (רבים לרבים)

```typescript
// Student יכול להירשם ל-Courses רבים
// Course יכול להיות בו Students רבים

@Table({ tableName: 'students' })
export class Student extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column
  name: string;

  @BelongsToMany(() => Course, () => StudentCourse)
  courses: Course[];
}

@Table({ tableName: 'courses' })
export class Course extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column
  title: string;

  @BelongsToMany(() => Student, () => StudentCourse)
  students: Student[];
}

// טבלת חיבור
@Table({ tableName: 'student_courses' })
export class StudentCourse extends Model {
  @ForeignKey(() => Student)
  @Column
  studentId: number;

  @ForeignKey(() => Course)
  @Column
  courseId: number;

  @Column
  grade: number; // ציון
}

// שימוש
const student = await Student.findByPk(1, {
  include: [Course]
});
console.log(student.courses); // כל הקורסים של הסטודנט

// הוספת קשר
const student = await Student.findByPk(1);
const course = await Course.findByPk(5);
await student.$add('course', course);

// הסרת קשר
await student.$remove('course', course);
```

### תרגיל עצמאי 5.1
צור מבנה של:
- `Author` (סופר)
- `Book` (ספר)
- כל סופר יכול לכתוב ספרים רבים
- כל ספר שייך לסופר אחד

<details>
<summary>פתרון</summary>

```typescript
@Table({ tableName: 'authors' })
export class Author extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column({ allowNull: false })
  name: string;

  @Column
  birthYear: number;

  @HasMany(() => Book)
  books: Book[];
}

@Table({ tableName: 'books' })
export class Book extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @ForeignKey(() => Author)
  @Column
  authorId: number;

  @BelongsTo(() => Author)
  author: Author;

  @Column({ allowNull: false })
  title: string;

  @Column
  publishYear: number;
}

// שימוש
const author = await Author.findByPk(1, {
  include: [Book]
});
console.log(`${author.name} wrote ${author.books.length} books`);
```
</details>

---

## תרגיל 6: Hooks

### מטרה
ללמוד איך להריץ קוד לפני/אחרי פעולות.

### דוגמה: הצפנת סיסמה

```typescript
import * as bcrypt from 'bcrypt';

@Table({ tableName: 'users' })
export class User extends Model {
  @Column
  email: string;

  @Column
  password: string;

  // לפני יצירה
  @BeforeCreate
  static async hashPassword(user: User) {
    if (user.password) {
      user.password = await bcrypt.hash(user.password, 10);
    }
  }

  // לפני עדכון
  @BeforeUpdate
  static async hashPasswordOnUpdate(user: User) {
    if (user.changed('password')) {
      user.password = await bcrypt.hash(user.password, 10);
    }
  }

  // אחרי יצירה
  @AfterCreate
  static async sendWelcomeEmail(user: User) {
    console.log(`Sending welcome email to ${user.email}`);
    // קוד שליחת מייל...
  }

  // לפני מחיקה
  @BeforeDestroy
  static async cleanupUserData(user: User) {
    console.log(`Cleaning up data for user ${user.id}`);
    // מחיקת קבצים, תמונות וכו'
  }
}
```

### סוגי Hooks נפוצים:

```typescript
// Create hooks
@BeforeCreate
@AfterCreate

// Update hooks
@BeforeUpdate
@AfterUpdate

// Delete hooks
@BeforeDestroy
@AfterDestroy

// Save hooks (create או update)
@BeforeSave
@AfterSave

// Validation hooks
@BeforeValidate
@AfterValidate

// Bulk operations
@BeforeBulkCreate
@AfterBulkCreate
@BeforeBulkUpdate
@AfterBulkUpdate
@BeforeBulkDestroy
@AfterBulkDestroy
```

### תרגיל עצמאי 6.1
צור מודל `Article` עם hooks:
1. לפני שמירה - המרת הכותרת לאותיות קטנות
2. אחרי יצירה - הדפסת הודעה
3. לפני מחיקה - בדיקה אם יש תגובות (אם כן, מנע מחיקה)

<details>
<summary>פתרון</summary>

```typescript
@Table({ tableName: 'articles' })
export class Article extends Model {
  @Column
  title: string;

  @Column(DataType.TEXT)
  content: string;

  @Column({ defaultValue: 0 })
  commentCount: number;

  @BeforeSave
  static normalizeTitle(article: Article) {
    if (article.title) {
      article.title = article.title.toLowerCase();
    }
  }

  @AfterCreate
  static logCreation(article: Article) {
    console.log(`New article created: "${article.title}"`);
  }

  @BeforeDestroy
  static preventDeleteIfComments(article: Article) {
    if (article.commentCount > 0) {
      throw new Error('Cannot delete article with comments');
    }
  }
}
```
</details>

---

## תרגיל 7: Transactions

### מטרה
ללמוד איך להשתמש בטרנזקציות להבטחת שלמות נתונים.

### מה זה Transaction?
Transaction מבטיח שכל הפעולות מצליחות או שהכל מתבטל.

### דוגמה: העברת כסף

```typescript
import { Sequelize } from 'sequelize-typescript';

// ללא transaction - מסוכן!
async function transferMoneyUnsafe(fromUserId: number, toUserId: number, amount: number) {
  const fromUser = await User.findByPk(fromUserId);
  const toUser = await User.findByPk(toUserId);

  fromUser.balance -= amount;
  await fromUser.save();

  // אם קורה שגיאה כאן, הכסף "נעלם"!

  toUser.balance += amount;
  await toUser.save();
}

// עם transaction - בטוח!
async function transferMoneySafe(
  sequelize: Sequelize,
  fromUserId: number,
  toUserId: number,
  amount: number
) {
  const t = await sequelize.transaction();

  try {
    const fromUser = await User.findByPk(fromUserId, { transaction: t });
    const toUser = await User.findByPk(toUserId, { transaction: t });

    if (fromUser.balance < amount) {
      throw new Error('Insufficient funds');
    }

    fromUser.balance -= amount;
    await fromUser.save({ transaction: t });

    toUser.balance += amount;
    await toUser.save({ transaction: t });

    await t.commit(); // מאשר את כל השינויים
    console.log('Transfer successful!');
  } catch (error) {
    await t.rollback(); // מבטל את כל השינויים
    console.error('Transfer failed:', error);
    throw error;
  }
}
```

### Transaction אוטומטי

```typescript
async function createUserWithProfile(userData: any, profileData: any) {
  return sequelize.transaction(async (t) => {
    // כל הפעולות כאן חלק מה-transaction
    const user = await User.create(userData, { transaction: t });
    
    profileData.userId = user.id;
    await Profile.create(profileData, { transaction: t });

    return user;
    // אם הכל עבר בהצלחה - commit אוטומטי
    // אם יש שגיאה - rollback אוטומטי
  });
}
```

### תרגיל עצמאי 7.1
כתוב פונקציה עם transaction שמבצעת הזמנה:
1. יוצרת Order
2. מפחיתה את ה-stock של המוצר
3. מוסיפה פריטים ל-OrderItems

<details>
<summary>פתרון</summary>

```typescript
async function createOrder(
  sequelize: Sequelize,
  userId: number,
  items: { productId: number; quantity: number }[]
) {
  return sequelize.transaction(async (t) => {
    // יצירת הזמנה
    const order = await Order.create(
      {
        userId,
        totalAmount: 0,
        status: 'pending',
      },
      { transaction: t }
    );

    let totalAmount = 0;

    // עבור כל פריט
    for (const item of items) {
      // מציאת המוצר
      const product = await Product.findByPk(item.productId, { transaction: t });

      if (!product) {
        throw new Error(`Product ${item.productId} not found`);
      }

      if (product.stock < item.quantity) {
        throw new Error(`Insufficient stock for ${product.name}`);
      }

      // הפחתת stock
      product.stock -= item.quantity;
      await product.save({ transaction: t });

      // יצירת OrderItem
      await OrderItem.create(
        {
          orderId: order.id,
          productId: product.id,
          quantity: item.quantity,
          price: product.price,
        },
        { transaction: t }
      );

      totalAmount += product.price * item.quantity;
    }

    // עדכון סכום ההזמנה
    order.totalAmount = totalAmount;
    await order.save({ transaction: t });

    return order;
  });
}
```
</details>

---

## תרגיל 8: Scopes

### מטרה
ללמוד איך ליצור שאילתות מוגדרות מראש.

### הגדרת Scopes

```typescript
@Table({
  tableName: 'users',
  scopes: {
    // Scope פעיל
    active: {
      where: { status: 'active' }
    },
    // Scope למבוגרים
    adults: {
      where: { age: { [Op.gte]: 18 } }
    },
    // Scope עם פרמטר
    olderThan: (age: number) => ({
      where: { age: { [Op.gt]: age } }
    }),
    // Scope מורכב
    activeWithProfile: {
      where: { status: 'active' },
      include: [{ model: Profile }]
    },
    // Default scope - תמיד פעיל
  },
  defaultScope: {
    attributes: { exclude: ['password'] } // תמיד מסתיר סיסמה
  }
})
export class User extends Model {
  @Column
  firstName: string;

  @Column
  age: number;

  @Column
  status: string;

  @Column
  password: string;
}
```

### שימוש ב-Scopes

```typescript
// שימוש ב-scope
const activeUsers = await User.scope('active').findAll();

// שימוש במספר scopes
const activeAdults = await User.scope(['active', 'adults']).findAll();

// scope עם פרמטר
const seniors = await User.scope({ method: ['olderThan', 60] }).findAll();

// ביטול default scope
const allUsersWithPassword = await User.unscoped().findAll();

// שילוב scope עם תנאים נוספים
const activeGmailUsers = await User.scope('active').findAll({
  where: { email: { [Op.like]: '%gmail%' } }
});
```

### Scope דינמי

```typescript
@Table({ tableName: 'products' })
export class Product extends Model {
  @Column
  name: string;

  @Column
  price: number;

  @Column
  category: string;

  // מתודה סטטית ליצירת scope דינמי
  static priceRange(minPrice: number, maxPrice: number) {
    return this.scope({
      where: {
        price: {
          [Op.between]: [minPrice, maxPrice]
        }
      }
    });
  }
}

// שימוש
const affordableProducts = await Product.priceRange(10, 100).findAll();
```

### תרגיל עצמאי 8.1
צור מודל `Post` עם scopes:
1. `published` - רק פוסטים שפורסמו
2. `draft` - רק טיוטות
3. `recent` - מ-30 הימים האחרונים
4. `withAuthor` - כולל את המחבר

<details>
<summary>פתרון</summary>

```typescript
@Table({
  tableName: 'posts',
  scopes: {
    published: {
      where: { status: 'published' }
    },
    draft: {
      where: { status: 'draft' }
    },
    recent: {
      where: {
        createdAt: {
          [Op.gte]: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)
        }
      }
    },
    withAuthor: {
      include: [{ model: User, as: 'author' }]
    }
  }
})
export class Post extends Model {
  @Column
  title: string;

  @Column(DataType.TEXT)
  content: string;

  @Column
  status: string;

  @ForeignKey(() => User)
  @Column
  authorId: number;

  @BelongsTo(() => User, 'authorId')
  author: User;
}

// שימוש
const recentPublishedPosts = await Post.scope(['published', 'recent', 'withAuthor']).findAll();
```
</details>

---

## תרגיל מסכם

צור מערכת של בלוג פשוטה עם:

1. **מודלים:**
   - `User` (משתמש)
   - `Post` (פוסט)
   - `Comment` (תגובה)
   - `Tag` (תגית)

2. **יחסים:**
   - משתמש יכול לכתוב פוסטים רבים
   - פוסט שייך למשתמש אחד
   - פוסט יכול להיות לו תגובות רבות
   - פוסט יכול להיות לו תגיות רבות (many-to-many)

3. **פיצ'רים:**
   - וולידציות על כל השדות
   - Hooks להצפנת סיסמה
   - Scopes לפוסטים מפורסמים/טיוטות
   - Transaction ליצירת פוסט עם תגיות

<details>
<summary>פתרון מלא</summary>

```typescript
// ========== User Model ==========
@Table({
  tableName: 'users',
  timestamps: true,
  defaultScope: {
    attributes: { exclude: ['password'] }
  },
  scopes: {
    withPassword: {
      attributes: { include: ['password'] }
    }
  }
})
export class User extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column({
    allowNull: false,
    validate: { len: [2, 50] }
  })
  name: string;

  @Column({
    allowNull: false,
    unique: true,
    validate: { isEmail: true }
  })
  email: string;

  @Column({
    allowNull: false,
    validate: { len: [6, 100] }
  })
  password: string;

  @HasMany(() => Post)
  posts: Post[];

  @HasMany(() => Comment)
  comments: Comment[];

  @BeforeCreate
  @BeforeUpdate
  static async hashPassword(user: User) {
    if (user.changed('password')) {
      user.password = await bcrypt.hash(user.password, 10);
    }
  }
}

// ========== Post Model ==========
@Table({
  tableName: 'posts',
  timestamps: true,
  scopes: {
    published: { where: { status: 'published' } },
    draft: { where: { status: 'draft' } },
    withAuthor: { include: [User] }
  }
})
export class Post extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @ForeignKey(() => User)
  @Column
  userId: number;

  @BelongsTo(() => User)
  author: User;

  @Column({
    allowNull: false,
    validate: { len: [5, 200] }
  })
  title: string;

  @Column({
    type: DataType.TEXT,
    allowNull: false
  })
  content: string;

  @Column({
    type: DataType.ENUM('draft', 'published'),
    defaultValue: 'draft'
  })
  status: string;

  @HasMany(() => Comment)
  comments: Comment[];

  @BelongsToMany(() => Tag, () => PostTag)
  tags: Tag[];
}

// ========== Comment Model ==========
@Table({
  tableName: 'comments',
  timestamps: true
})
export class Comment extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @ForeignKey(() => Post)
  @Column
  postId: number;

  @BelongsTo(() => Post)
  post: Post;

  @ForeignKey(() => User)
  @Column
  userId: number;

  @BelongsTo(() => User)
  author: User;

  @Column({
    type: DataType.TEXT,
    allowNull: false,
    validate: { len: [1, 1000] }
  })
  content: string;
}

// ========== Tag Model ==========
@Table({
  tableName: 'tags',
  timestamps: true
})
export class Tag extends Model {
  @Column({ primaryKey: true, autoIncrement: true })
  id: number;

  @Column({
    allowNull: false,
    unique: true,
    validate: { len: [2, 30] }
  })
  name: string;

  @BelongsToMany(() => Post, () => PostTag)
  posts: Post[];
}

// ========== PostTag (Join Table) ==========
@Table({ tableName: 'post_tags', timestamps: false })
export class PostTag extends Model {
  @ForeignKey(() => Post)
  @Column
  postId: number;

  @ForeignKey(() => Tag)
  @Column
  tagId: number;
}

// ========== Service Example ==========
export class BlogService {
  async createPostWithTags(
    sequelize: Sequelize,
    postData: any,
    tagNames: string[]
  ) {
    return sequelize.transaction(async (t) => {
      // יצירת הפוסט
      const post = await Post.create(postData, { transaction: t });

      // מציאה או יצירת תגיות
      const tags = await Promise.all(
        tagNames.map(name =>
          Tag.findOrCreate({
            where: { name },
            transaction: t
          })
        )
      );

      // חיבור התגיות לפוסט
      await post.$set('tags', tags.map(([tag]) => tag), { transaction: t });

      return post;
    });
  }

  async getPublishedPostsWithDetails() {
    return Post.scope(['published', 'withAuthor']).findAll({
      include: [
        { model: Comment, include: [User] },
        { model: Tag }
      ],
      order: [['createdAt', 'DESC']]
    });
  }
}
```
</details>

---

## טיפים נוספים

### 1. Raw Queries
לפעמים צריך SQL ישיר:

```typescript
const [results] = await sequelize.query(
  'SELECT * FROM users WHERE age > :age',
  {
    replacements: { age: 25 },
    type: QueryTypes.SELECT
  }
);
```

### 2. Virtual Fields
שדות מחושבים:

```typescript
@Column({
  type: DataType.VIRTUAL,
  get() {
    return `${this.getDataValue('firstName')} ${this.getDataValue('lastName')}`;
  }
})
fullName: string;
```

### 3. Indexes
לביצועים טובים יותר:

```typescript
@Table({
  indexes: [
    { fields: ['email'] },
    { unique: true, fields: ['username'] },
    { fields: ['firstName', 'lastName'] }
  ]
})
```

---

## משאבים נוספים

- [Sequelize Documentation](https://sequelize.org/)
- [Sequelize TypeScript](https://github.com/sequelize/sequelize-typescript)

**בהצלחה בתרגול! 💪**
