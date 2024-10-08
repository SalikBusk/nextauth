# Trin-for-Trin Guide til Integration af NextAuth

Dette GitHub repository indeholder en omfattende trin-for-trin vejledning til opsætning af NextAuth i et Next.js-projekt. NextAuth er en kraftfuld og fleksibel autentificeringsløsning, der understøtter flere loginmetoder, herunder OAuth, e-mail-login og brugerdefinerede autentificeringsstrategier. Denne guide er designet til at gøre det nemt for dig at integrere sikker login-funktionalitet i din applikation.

---

## Opsætning af LoginForm og RegisterForm

### Card Wrapper Komponent

For at strukturere vores login- og registreringsformularer vil vi først oprette en **CardWrapper** komponent, som kan bruges til at indpakke vores formularindhold og give et ensartet design.

#### Kode til CardWrapper Komponent (`/components/auth/cardWrapper.tsx`)

```typescript
"use client"

import React, { ReactNode } from 'react';
import { Card, CardContent, CardFooter, CardHeader } from '../ui/card';
import { BackButton } from './BackButton';
import { Header } from './Header';

interface CardWrapperProps {
    children: ReactNode;
    headerTitle: string;
    headerLabel: string;
    backButtonLabel: string;
    backButtonHref: string;
}

const CardWrapper = ({
    children,
    headerTitle,
    headerLabel,
    backButtonLabel,
    backButtonHref,
}: CardWrapperProps) => {
    return (
        <Card className='w-[400px] shadow-md'>
            <CardHeader>
                <Header title={headerTitle} label={headerLabel} />
            </CardHeader>
            <CardContent>
                {children}
            </CardContent>
            <CardFooter>
                <BackButton
                    label={backButtonLabel}
                    href={backButtonHref}
                />
            </CardFooter>
        </Card>
    );
}

export default CardWrapper;
```

### Oprettelse af Schemas

For at validere brugerens input ved login og registrering vil vi oprette schemas ved hjælp af **Zod**. Zod er et bibliotek til runtime data validation, der gør det nemt at definere og validere objekter.

#### Installation af nødvendige pakker

Først skal vi installere Zod:

```bash
npm install zod
```

#### Kode til Schemas (`@/schemas/index.ts`)

```javascript
import * as z from "zod";

export const LoginSchema = z.object({
    email: z.string().email({
        message: "Email er påkrævet",
    }),
    password: z.string().min(1, {
        message: "Adgangskode er påkrævet",
    }),
});

export const RegisterSchema = z.object({
    email: z.string().email({
        message: "Email er påkrævet",
    }),
    password: z.string().min(6, {
        message: "Minimum 6 tegn kræves",
    }),
    name: z.string().min(1, {
        message: "Navn er påkrævet",
    }),
});
```

### Opsætning af Login- og Registreringshandlinger

Nu skal vi implementere funktionaliteterne til login og registrering. Vi opretter to separate filer til handlingerne: `login.ts` og `register.ts`.

#### Login Handling (`@/actions/login.ts`)

```javascript
"use server"

import * as z from "zod";
import { LoginSchema } from "../schemas";

export const login = async (values: z.infer<typeof LoginSchema>) => {
    const validatedFields = LoginSchema.safeParse(values);

    if (!validatedFields.success) {
        return { error: "Ugyldige felter!" };
    }
    
    // Logik for at håndtere login
    // Dette kunne inkludere opkald til en backend eller database.
    return { success: "Login succesfuldt!", };
}
```

#### Registrerings Handling (`@/actions/register.ts`)

```javascript
"use server"

import * as z from "zod";
import { RegisterSchema } from "../schemas";

export const register = async (values: z.infer<typeof RegisterSchema>) => {
    const validatedFields = RegisterSchema.safeParse(values);

    if (!validatedFields.success) {
        return { error: "Ugyldige felter!" };
    }
    
    // Logik for at håndtere registrering
    // Dette kunne inkludere opkald til en backend for at oprette en bruger.
    return { success: "Bekræftelses-e-mail sendt!", };
}
```

---

## Opsætning af LoginForm og RegisterForm 

### LoginForm (`@components/auth/LoginForm.tsx`)

```javascript
"use client";

import * as z from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

import { Input } from "@/components/ui/input";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,  
} from "@/components/ui/form";
import { Button } from "@/components/ui/button";
import { FormError } from "@/components/form-error";
import { FormSuccess } from "@/components/form-success";
import CardWrapper from "./CardWrapper";
import { LoginSchema } from "../../../schemas";
import { login } from "../../../actions/login";
import { useState, useTransition } from "react";

export const LoginForm = () => {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | undefined>("");
  const [success, setSuccess] = useState<string | undefined>("");

  const form = useForm<z.infer<typeof LoginSchema>>({
    resolver: zodResolver(LoginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  const onSubmit = (values: z.infer<typeof LoginSchema>) => {
    setError("");
    setSuccess("");
    
    startTransition(() => {
      login(values)
      .then((data) => {
        setError(data?.error);
        setSuccess(data?.success);
    })
    });
  };

  return (
    <CardWrapper
      headerTitle="Login"
      headerLabel="Velkommen tilbage"
      backButtonLabel="Har du ikke en konto?"
      backButtonHref="/auth/register"
    >
      <Form {...form}>
        <form 
          onSubmit={form.handleSubmit(onSubmit)}
          className="space-y-6"
        >
          <div className="space-y-4">
            <FormField
                  control={form.control}
                  name="email"
                  render={({ field }) => (
                    <FormItem>
                      <FormLabel>Email</FormLabel>
                      <FormControl>
                        <Input
                          {...field}
                          disabled={isPending}
                          placeholder="john.doe@example.com"
                          type="email"
                        />
                      </FormControl>
                      <FormMessage />
                    </FormItem>
                  )}
            />
            <FormField
                  control={form.control}
                  name="password"
                  render={({ field }) => (
                    <FormItem>
                      <FormLabel>Adgangskode</FormLabel>
                      <FormControl>
                        <Input
                          {...field}
                          disabled={isPending}
                          placeholder="******"
                          type="password"
                        />
                      </FormControl>
                      
                      <FormMessage />
                    </FormItem>
                  )}
                />
          </div>
          <FormError message={error} />
          <FormSuccess message={success} />
          <Button
            disabled={isPending}
            type="submit"
            className="w-full"
          >
            Login
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};
```

### RegisterForm (`@components/auth/RegisterForm.tsx`)

```javascript
"use client";

import * as z from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

import { Input } from "@/components/ui/input";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,  
} from "@/components/ui/form";
import { Button } from "@/components/ui/button";
import { FormError } from "@/components/form-error";
import { FormSuccess } from "@/components/form-success";
import CardWrapper from "./CardWrapper";
import { RegisterSchema } from "../../../schemas";
import { register } from "../../../actions/register";
import { useState, useTransition } from "react";

export const RegisterForm = () => {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | undefined>("");
  const [success, setSuccess] = useState<string | undefined>("");

  const form = useForm<z.infer<typeof RegisterSchema>>({
    resolver: zodResolver(RegisterSchema),
    defaultValues: {
      email: "",
      password: "",
      name: "",
    },
  });

  const onSubmit = (values: z.infer<typeof RegisterSchema>) => {
    setError("");
    setSuccess("");
    
    startTransition(() => {
      register(values)
      .then((data) => {
        setError(data?.error);
        setSuccess(data?.success);
    })
    });
  };

  return (
    <CardWrapper
      headerTitle="Tilmeld dig"
      headerLabel="Opret en konto"
      backButtonLabel="Har du allerede en konto?"
      backButtonHref="/auth/login"
    >
      <Form {...form}>
        <form 
          onSubmit={

form.handleSubmit(onSubmit)}
          className="space-y-6"
        >
          <div className="space-y-4">
            <FormField
                  control={form.control}
                  name="name"
                  render={({ field }) => (
                    <FormItem>
                      <FormLabel>Navn</FormLabel>
                      <FormControl>
                        <Input
                          {...field}
                          disabled={isPending}
                          placeholder="John Doe"
                          type="text"
                        />
                      </FormControl>
                      <FormMessage />
                    </FormItem>
                  )}
            />
            <FormField
                  control={form.control}
                  name="email"
                  render={({ field }) => (
                    <FormItem>
                      <FormLabel>Email</FormLabel>
                      <FormControl>
                        <Input
                          {...field}
                          disabled={isPending}
                          placeholder="john.doe@example.com"
                          type="email"
                        />
                      </FormControl>
                      <FormMessage />
                    </FormItem>
                  )}
            />
            <FormField
                  control={form.control}
                  name="password"
                  render={({ field }) => (
                    <FormItem>
                      <FormLabel>Adgangskode</FormLabel>
                      <FormControl>
                        <Input
                          {...field}
                          disabled={isPending}
                          placeholder="******"
                          type="password"
                        />
                      </FormControl>
                      <FormMessage />
                    </FormItem>
                  )}
            />
          </div>
          <FormError message={error} />
          <FormSuccess message={success} />
          <Button
            disabled={isPending}
            type="submit"
            className="w-full"
          >
            Tilmeld dig
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};
```

---

## Afslutning

Med denne guide har du nu en grundlæggende opsætning for at implementere login- og registreringsfunktioner i dit Next.js-projekt ved hjælp af NextAuth. Det inkluderer et struktureret design, data validering og håndtering af serveranmodninger. Du kan nu tilpasse og udvide denne opsætning baseret på dine specifikke krav og designpræferencer.
