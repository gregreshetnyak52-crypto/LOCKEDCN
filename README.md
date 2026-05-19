# LOCKEDCN

Частный люксовый сервис закупок из Китая и Европы.

## Стек

- Статический HTML/CSS/JS (без сборщиков)
- [Supabase](https://supabase.com) — БД, авторизация, Storage, real-time чат
- [EmailJS](https://emailjs.com) — email-уведомления о новых заказах

## Структура файлов

```
index.html      — лендинг + форма заказа + авторизация
cabinet.html    — личный кабинет клиента (заказы, профиль, чат)
admin.html      — панель менеджера (все заказы, чат с клиентами)
privacy.html    — политика конфиденциальности
images/         — фото товаров (dior/, balenciaga/, Ф1 мерч/)
videos/         — видео-примеры заказов
```

## Настройка Supabase

### Таблицы

```sql
create table profiles (
  id uuid references auth.users primary key,
  full_name text,
  telegram text,
  email text
);

create table orders (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users,
  link text,
  name text,
  size text,
  color text,
  qty int default 1,
  telegram text,
  comment text,
  file_url text,
  status text default 'new',
  created_at timestamptz default now()
);

create table messages (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users,
  content text,
  from_admin boolean default false,
  created_at timestamptz default now()
);
```

### RPC-функция для проверки администратора

```sql
create or replace function is_current_user_admin()
returns boolean language sql security definer as $$
  select exists (
    select 1 from auth.users
    where id = auth.uid()
    and raw_user_meta_data->>'is_admin' = 'true'
  );
$$;
```

### Назначить администратора

Dashboard → Authentication → Users → найти пользователя → Edit → Raw User Meta Data:
```json
{ "is_admin": true }
```

### Storage для скриншотов заказов

Dashboard → Storage → New bucket → Name: `order-files` → Public: ✓

## Настройка EmailJS

1. Зарегистрироваться на [emailjs.com](https://emailjs.com)
2. Создать Email Service
3. Создать шаблон с переменными: `order_link`, `order_name`, `order_size`, `order_color`, `order_qty`, `order_tg`, `order_comment`, `order_file`
4. Вставить ключи в `index.html` (строки ~728–730):

```js
const EMAILJS_PUBLIC_KEY = 'ваш_public_key';
const EMAILJS_SERVICE_ID = 'ваш_service_id';
const EMAILJS_TEMPLATE_ID = 'ваш_template_id';
```

## Деплой на Vercel

Подключить GitHub-репозиторий через vercel.com — деплой происходит автоматически при каждом push в `main`. Или через CLI:

```bash
npm i -g vercel
vercel --prod
```
