# Component-library-with-typescript

## Window width context provider:
```bash
"use client";

import { createContext, useEffect, useState, ReactNode } from "react";

type TContextProvider = {
  windowWidth: number;
};

export const ContextProvider = createContext<TContextProvider | null>(null);

const MyContextProvider = ({ children }: {children: ReactNode}) => {
  const [windowWidth, setWindowWidth] = useState<number | null>(null);

  useEffect(() => {
    if (typeof window !== "undefined") {
      setWindowWidth(window.innerWidth); // Set initial window width

      const handleWindowResize = () => {
        setWindowWidth(window.innerWidth);
      };

      window.addEventListener("resize", handleWindowResize);

      return () => {
        window.removeEventListener("resize", handleWindowResize);
      };
    }
  }, []);

  const infoProvider: TContextProvider | null = windowWidth !== null ? { windowWidth } : null;

  return (
    <ContextProvider.Provider value={infoProvider}>
      {children}
    </ContextProvider.Provider>
  );
};

export default MyContextProvider;
```

## Dynamic heading:
```bash
"use client";

import React, { HtmlHTMLAttributes, useContext } from "react";
import clsx from "clsx";
import { ContextProvider } from "@/lib/MyContextProvider";

interface HeaderProps extends HtmlHTMLAttributes<HTMLHeadElement> {
  color?: "white" | "primary" | "black";
  center?: boolean;
  level?: 1 | 2 | 3 | 4 | 5;
  children?: React.ReactNode;
  className?: string;
}

const Heading: React.FC<HeaderProps> = ({
  color = "black",
  center = false,
  level = 2,
  children,
  className,
  ...rest
}) => {
  const textColor = {
    white: "text-white",
    primary: "text-primary",
    black: "text-header",
  }[color];

  const context = useContext(ContextProvider);
  const windowWidth = context ? context.windowWidth : 0;
  const isSmallScreen = windowWidth < 500;

  const finalLevel = isSmallScreen
    ? level + 1 > 5
      ? 5
      : ((level + 1) as typeof level)
    : (level as typeof level);

  const textSize = {
    1: "text-[75px] font-bold",
    2: "text-[46px] font-semibold",
    3: "text-[28px] font-medium",
    4: "text-[17px] font-normal",
    5: "text-[10px] font-light",
  }[finalLevel];

  const Tag: React.ElementType = `h${finalLevel}`;

  return (
    <header className={clsx("py-4 px-6", center && "text-center")}>
      <Tag className={clsx(textSize, textColor, className)} {...rest}>
        {children}
      </Tag>
    </header>
  );
};

export default Heading;
```
