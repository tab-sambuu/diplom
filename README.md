# 🎨 Гар урлал дэлгүүр (Handmade Shop)

Монголын гар урлалчдын бүтээгдэхүүнийг худалдан авах, борлуулах платформ.

## 📋 Технологи

### Backend

- **Node.js** + **TypeScript**
- **Apollo Server** (GraphQL)
- **Prisma ORM**
- **MySQL** (Docker)
- **JWT** authentication
- **bcrypt** password hashing

### Frontend

- **React 18** + **TypeScript**
- **Vite** (bundler)
- **TailwindCSS** (styling)
- **Apollo Client** (GraphQL client)
- **React Router** (routing)

## 🚀 Эхлүүлэх заавар

### 1. Database эхлүүлэх (Docker)

```bash
cd back
docker-compose up -d
```

Энэ нь MySQL болон Adminer-ийг эхлүүлнэ:

- MySQL: `localhost:3306`
- Adminer: `http://localhost:8080`

### 2. Backend тохируулах

```bash
cd back

# Dependencies суулгах
npm install

# .env файл үүсгэх
cp .env.example .env
# .env файлыг засаж DATABASE_URL, JWT_SECRET оруулна уу

# Prisma generate & migrate
npx prisma generate
npx prisma migrate dev --name init

# Seed data оруулах
npm run prisma:seed

# Backend эхлүүлэх
npm run dev
```

Backend `http://localhost:4000/graphql` дээр ажиллана.

### 3. Frontend тохируулах

```bash
cd web

# Dependencies суулгах
npm install

# Frontend эхлүүлэх
npm run dev
```

Frontend `http://localhost:3000` дээр ажиллана.

## 📦 Prisma commands

```bash
# Prisma Client үүсгэх
npx prisma generate

# Database migration үүсгэх
npx prisma migrate dev --name migration_name

# Seed data оруулах
npm run prisma:seed

# Prisma Studio ашиглах (database UI)
npm run prisma:studio
```

## 👤 Demo бүртгэлүүд

Seed дараах бүртгэлүүдийг үүсгэнэ:

| И-мэйл            | Нууц үг     | Хандах эрх                        |
| ----------------- | ----------- | --------------------------------- |
| buyer@example.mn  | password123 | Худалдан авагч (wallet: 500,000₮) |
| saruul@example.mn | password123 | Худалдагч                         |
| oyunaa@example.mn | password123 | Худалдагч                         |
| boldoo@example.mn | password123 | Худалдагч                         |
| admin@handmade.mn | admin123    | Админ                             |

## 🔑 Үндсэн функцууд

### ✅ Хэрэгжүүлсэн

- ✅ Бүртгэл / Нэвтрэх (JWT)
- ✅ Бүтээгдэхүүн CRUD (худалдагч)
- ✅ Бүтээгдэхүүн жагсаалт + хайлт + шүүлт
- ✅ Бүтээгдэхүүн дэлгэрэнгүй
- ✅ Сагс (localStorage)
- ✅ Wallet (цэнэглэлт + үлдэгдэл)
- ✅ purchaseWithWallet (atomic transaction)
- ✅ Захиалга түүх
- ✅ Худалдагчийн самбар
- ✅ Админ зөвшөөрөл (product status)

### 🔮 Ирээдүйн боломжууд

- Reviews & ratings
- Seller payout system
- Multi-image gallery
- Real payment (Stripe webhook)
- Admin dashboard
- Email notifications
- SMS notifications

## 🧪 Тест кейсүүд

### 1. Wallet Top-up (GraphQL Playground)

```graphql
mutation {
  topUpFake(amount: 100000) {
    id
    balance
  }
}
```

**Expected:** Wallet balance 100,000₮-ээр нэмэгдэнэ, transaction үүснэ.

### 2. Purchase with Wallet

```graphql
mutation {
  purchaseWithWallet(
    input: {
      items: [{ productId: 1, quantity: 2 }]
      shippingAddress: "УБ, СБД, 1-р хороо"
    }
  ) {
    success
    message
    order {
      id
      totalAmount
      status
    }
  }
}
```

**Expected:**

- Balance хүрэлцэхгүй бол: `success: false, message: "Wallet үлдэгдэл хүрэлцэхгүй байна"`
- Амжилттай бол: order үүсч, balance бууралт, transaction үүснэ.

### 3. Concurrency Test (2 хэрэглэгч зэрэг худалдан авалт)

Postman эсвэл k6 ашиглан 2 request зэрэг илгээх:

```bash
# Terminal 1
curl -X POST http://localhost:4000/graphql \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { purchaseWithWallet(input: {items: [{productId: 1, quantity: 1}], shippingAddress: \"Test\"}) { success message } }"}'

# Terminal 2 (зэрэг ажиллуулах)
curl -X POST http://localhost:4000/graphql \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { purchaseWithWallet(input: {items: [{productId: 1, quantity: 1}], shippingAddress: \"Test\"}) { success message } }"}'
```

**Expected:** Prisma transaction-ын ачаар нэг нь амжилттай, нөгөө нь "balance хүрэлцэхгүй" гэх алдаа өгнө.

### 4. Product Stock Check

Бүтээгдэхүүн stock-оос илүү тоо захиалахыг оролдох:

```graphql
mutation {
  purchaseWithWallet(
    input: { items: [{ productId: 1, quantity: 999 }], shippingAddress: "Test" }
  ) {
    success
    message
  }
}
```

**Expected:** `success: false, message: "Бүтээгдэхүүн хүрэлцэхгүй байна"`

### 5. Guest Checkout (wallet-гүй)

Frontend дээр нэвтрэлгүй бүтээгдэхүүн сагсанд хийж checkout дарах.

**Expected:** "/login" руу redirect хийнэ.

## 📊 Database Schema

Үндсэн моделууд:

- **User** — хэрэглэгч (BUYER, SELLER, ADMIN)
- **Profile** — нэмэлт мэдээлэл
- **Wallet** — түрийвч (balance: BigInt)
- **WalletTransaction** — гүйлгээний түүх (immutable audit log)
- **Product** — бүтээгдэхүүн (price: BigInt)
- **Order** — захиалга
- **OrderItem** — захиалгын бүтээгдэхүүн
- **Category** — ангилал
- **Review** — үнэлгээ

## 💰 Wallet Flow

1. **Top-up:** `topUpFake(amount)` → balance increment + transaction (TOP_UP)
2. **Purchase:** `purchaseWithWallet(input)` → atomic:
   - Check balance >= totalAmount
   - Create order + order items
   - Decrement product stock
   - Decrement wallet balance
   - Create transaction (PURCHASE, negative amount)
3. **Refund (future):** Admin refunds order → increment balance + REFUND transaction

## 🔐 Security

- Passwords: **bcrypt** hashed (10 rounds)
- Auth: **JWT** (7 days expiry)
- Token storage: localStorage (frontend)
- Authorization: Context-based (me query)
- SQL Injection: **Prisma** ORM автоматаар хамгаална

## 📝 .env тохиргоо

### Backend (`back/.env`)

```env
DATABASE_URL="mysql://root:rootpassword@localhost:3306/handmade_shop"
JWT_SECRET="your-super-secret-jwt-key-change-in-production"
PORT=4000
NODE_ENV=development
```

## 🎯 Диплом тайлбарт оруулах зүйлс

1. **Архитектур диаграм:** Frontend (React) ↔ GraphQL API ↔ Prisma ↔ MySQL
2. **ER диаграм:** Prisma schema-аас үүсгэнэ (`prisma studio` screenshot)
3. **Wallet flow diagram:** topUpFake → purchaseWithWallet (sequence diagram)
4. **Security considerations:** JWT, bcrypt, HTTPS (production)
5. **Testing:** Manual tests + screenshots (GraphQL Playground, Frontend)
6. **Concurrency handling:** Prisma transaction ашигласан тайлбар
7. **Future improvements:** Stripe, email notifications, review moderation

## 🐛 Troubleshooting

### MySQL холбогдохгүй байна

```bash
# Docker container ажиллаж байгаа эсэхийг шалгах
docker ps

# Container дахин эхлүүлэх
docker-compose down
docker-compose up -d
```

### Prisma migration алдаа

```bash
# Database reset (АНХААРУУЛГА: бүх өгөгдөл устана!)
npx prisma migrate reset

# Дахин seed оруулах
npm run prisma:seed
```

### Frontend Apollo Client алдаа

GraphQL endpoint зөв эсэхийг шалгах: `http://localhost:4000/graphql`

Backend ажиллаж байгаа эсэхийг `/health` endpoint-оор шалгах:

```bash
curl http://localhost:4000/health
```

## 📚 Нэмэлт материал

- [Prisma Docs](https://www.prisma.io/docs)
- [Apollo Server Docs](https://www.apollographql.com/docs/apollo-server)
- [Apollo Client Docs](https://www.apollographql.com/docs/react)
- [TailwindCSS Docs](https://tailwindcss.com/docs)

## 👨‍💻 Хөгжүүлэгч

**Sambuu** — Диплом төсөл 2024

---

Амжилт хүсье! 🚀
