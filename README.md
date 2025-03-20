# Penanganan Input API Pengguna Menggunakan Next.js, bcrypt-ts, dan TypeScript

Berikut adalah panduan lengkap untuk menangani input API pengguna dengan Next.js, bcrypt-ts, dan TypeScript.

## 1. Setup Proyek Next.js dengan TypeScript
```bash
npx create-next-app@latest --ts
cd your-project
```

## 2. Instalasi depedensi bcrypt-ts
```bash
npm install bcrypt-ts
```

## 3. Struktur direktori
```
├── app/
│   └── api/
│       └── auth/
│           └── register.ts
└── .env
```

## 4. Definisi Tipe Data(TypeScript)
```ts
export interface UserRegistration {
  username: string;
  email: string;
  password: string;
}
```
## 5. Validasi Input

```ts
import {z} from 'zod';
export const registrationSchema = z.object({
  username: z.string().min(3),
  email: z.string().email(),
  password: z.string().min(8),
})

```
## 6. Implementasi API Route

```ts
// pages/api/auth/register.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { hash } from 'bcrypt-ts';
import { registrationSchema } from '../../../lib/validation';
import { UserRegistration } from '../../../types/user';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    // 1. Validasi Input
    const validatedData = registrationSchema.parse(req.body);
    
    // 2. Hash Password
    const hashedPassword = await hash(validatedData.password, 12);
    
    // 3. Simpan ke Database (Contoh)
    const userData: UserRegistration = {
      ...validatedData,
      password: hashedPassword
    };
    
    // 4. Simpan userData ke database di sini
    
    return res.status(201).json({ message: 'User created successfully' });
    
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        message: 'Validation error',
        errors: error.errors
      });
    }
    
    console.error(error);
    return res.status(500).json({ message: 'Internal server error' });
  }
}

```

# 7. Keamanan dengan bcrypt-ts
Hashing Password

```ts
const saltRounds = 12;
const hashedPassword = await hash(plainPassword, saltRounds);
```
Verifikasi Password
```ts
import { compare } from 'bcrypt-ts';

const isPasswordValid = await compare(inputPassword, storedHash);
```
## 8. Error Handling
- 400 Bad Request: Untuk kesalahan validasi input
- 405 Method Not Allowed: Untuk method HTTP yang tidak didukung
- 500 Internal Server Error: Untuk kesalahan server yang tidak terduga

## 9. Penggunaan di Frontend

```ts
// Contoh fetch request dari client
const registerUser = async (userData: UserRegistration) => {
  try {
    const response = await fetch('/api/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });

    if (!response.ok) {
      throw new Error('Registration failed');
    }
    
    return await response.json();
  } catch (error) {
    console.error('Registration error:', error);
    throw error;
  }
};
```

## 10. Noted
10. Best Practices
    - Gunakan HTTPS: Pastikan aplikasi di-deploy dengan HTTPS
    - Rate Limiting: Implementasi rate limiting untuk mencegah brute force
    - Sanitasi Input: Bersihkan input dari potensi XSS
    - Logging: Catat aktivitas penting tanpa menyimpan informasi sensitif
    - Environment Variables: Simpan salt rounds di .env
   
```env
# .env
BCRYPT_SALT_ROUNDS=12
```

## 11. Testing dengan Postman
request
```postman
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```
 response
```postman
{
  "message": "User created successfully"
}
```



