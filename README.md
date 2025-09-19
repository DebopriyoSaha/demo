export type User = {
  id: string;
  firstName: string;
  lastName: string;
  email: string;   // stored lowercased for matching
  password: string; // DEMO ONLY — plain text. In real apps, hash this.
  phone: string;
};

const USERS_KEY = "journal.users";        // array<User>
const SESSION_KEY = "journal.currentUser"; // user id

function loadUsers(): User[] {
  try {
    const raw = localStorage.getItem(USERS_KEY);
    return raw ? (JSON.parse(raw) as User[]) : [];
  } catch {
    return [];
  }
}

function saveUsers(users: User[]) {
  localStorage.setItem(USERS_KEY, JSON.stringify(users));
}

export function signup(user: Omit<User, "id">): User {
  const users = loadUsers();
  const emailL = user.email.toLowerCase();
  if (users.some(u => u.email === emailL)) {
    throw new Error("That email is already in use.");
  }
  const newUser: User = { ...user, id: crypto.randomUUID(), email: emailL };
  users.push(newUser);
  saveUsers(users);
  localStorage.setItem(SESSION_KEY, newUser.id); // sign in after signup
  return newUser;
}

export function login(email: string, password: string): User {
  const users = loadUsers();
  const emailL = email.toLowerCase();
  const found = users.find(u => u.email === emailL && u.password === password);
  if (!found) throw new Error("Invalid email or password.");
  localStorage.setItem(SESSION_KEY, found.id);
  return found;
}

export function getCurrentUser(): User | null {
  const id = localStorage.getItem(SESSION_KEY);
  if (!id) return null;
  const users = loadUsers();
  return users.find(u => u.id === id) || null;
}

export function logout() {
  localStorage.removeItem(SESSION_KEY);
}
