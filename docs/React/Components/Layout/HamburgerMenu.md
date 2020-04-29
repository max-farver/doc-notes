---
name: Hamburger Menu
route: /react/hamburger-menu
menu: React - Components - Layout
---

# Hamburger Menu

Made of of the button and the menu.

_*NOTE: These use NextJS, so they have the Link and Router usage, just remove/replace for other frameworks.*_

## Menu

```javascript
import React, { useRef, useEffect } from "react"
import { motion } from "framer-motion"
import Link from "next/link"
import { useRouter } from "next/router"
import { useAppState } from "../../../utils/appContext"

const variants = {
  open: { x: 0 },
  closed: { x: "56rem" },
}

const MobileMenu = ({ isShowing, toggle, navLinks }) => {
  const { user, setUser } = useAppState()
  const router = useRouter()
  const logoutUser = async () => {
    await fetch("/api/auth/logout", {
      method: "POST",
    })
    setUser("")
    router.reload()
  }

  const menuRef = useRef()
  const handleOutsideClick = e => {
    if (isShowing && menuRef.current && !menuRef.current.contains(e.target)) {
      toggle()
    }
  }

  useEffect(() => {
    document.addEventListener("click", handleOutsideClick)

    return () => {
      document.removeEventListener("click", handleOutsideClick)
    }
  })

  return (
    <motion.div
      className="absolute top-0 right-0 w-56  bg-primary-900 h-screen -z-10"
      animate={isShowing ? "open" : "closed"}
      initial={{ x: "56rem" }}
      variants={variants}
      transition={{ stiffness: 1, duration: 0.3 }}
      ref={menuRef}
    >
      <ul className="flex flex-col pt-16 px-4 text-gray-50">
        {navLinks.map(link => (
          <Link key={`${link.name}_${link.path}_mobile`} href={link.path}>
            <a className="mb-4 px-2 py-2">{link.name}</a>
          </Link>
        ))}
        <div className="border-b mb-4"></div>
        {user ? (
          <li key={`$user_nav_desktop`}>
            <button onClick={logoutUser} className="inline px-2 py-2">
              Logout
            </button>
          </li>
        ) : (
          <Link key={`$user_nav_desktop`} href="login">
            <a className="px-2 py-2">Login</a>
          </Link>
        )}
      </ul>
    </motion.div>
  )
}

export default MobileMenu
```

## Button

```javascript
import React from "react"
import { useToggle } from "../../../utils/hooks"
import { motion } from "framer-motion"

const MobileMenuButton = ({ isShowing, toggle }) => {
  return (
    <button
      name={isShowing ? "close menu" : "open menu"}
      onClick={!isShowing && toggle}
      className="self-center focus:outline-none z-10"
    >
      <svg height="24" width="24">
        <motion.line
          initial={{
            y2: 4,
          }}
          animate={{
            y2: isShowing ? 20 : 4,
          }}
          transition={{ duration: 0.2 }}
          x1="4"
          x2="20"
          y1="4"
          stroke="#F0B428"
          strokeWidth="3"
          strokeLinecap="round"
        />
        <motion.line
          initial={{
            opacity: 1,
          }}
          animate={{
            opacity: isShowing ? 0 : 1,
          }}
          transition={{ duration: 0.2 }}
          x1="4"
          x2="20"
          y1="12"
          y2="12"
          stroke="#F0B428"
          strokeWidth="3"
          strokeLinecap="round"
        />
        <motion.line
          initial={{
            y2: 20,
          }}
          animate={{
            y2: isShowing ? 4 : 20,
          }}
          transition={{ duration: 0.2 }}
          x1="4"
          x2="20"
          y1="20"
          stroke="#F0B428"
          strokeWidth="3"
          strokeLinecap="round"
        />
      </svg>
    </button>
  )
}

export default MobileMenuButton
```

## Usage

```javascript
const Nav = () => {
  const windowSize = useWindowSize()
  const { isToggled: isShowing, toggle } = useToggle(false)

  return (
    <div className="bg-primary-900 fixed top-0 w-full border-b-2 border-secondary-500 z-10">
      <nav className="max-w-6xl mx-auto flex justify-between items-baseline px-4 text-gray-50">
        <Link href="/">
          <a className="tracking-wide sm:tracking-normal text-xl sm:text-2xl font-display py-2">
            {windowSize.width > 500
              ? "Northwest Financial Consulting"
              : "NW Financial Consulting"}
          </a>
        </Link>
        {windowSize?.width > 1000 ? (
          <DesktopMenu navLinks={navLinks} />
        ) : (
          // ##########################
          <>
            <MobileMenuButton isShowing={isShowing} toggle={toggle} />
            <MobileMenu
              navLinks={navLinks}
              toggle={toggle}
              isShowing={isShowing}
            />
          </>
          // #########################
        )}
      </nav>
    </div>
  )
}
```

**This could easily be made a single self-contained component, I just don't want to right this moment but want to be able to look back easily and do it later.**
