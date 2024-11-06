```bash
// .bashrc
alias vt='~/vite-project.sh'
```



```bash
#!/bin/bash

# Default template
template="react-swc-ts"
projectName=""
declare -a packages_installed

show_help() {
  echo "Usage: setup-project.sh [options]"
  echo ""
  echo "  -N               Set the project name (required)"
  echo "  -T, --template   Specify the project template (default: react-swc-ts)"
  echo "  -P, --plugin     Specify plugins to install"
  echo ""
  echo "Package Options:"
  echo "  -a, --antd       Install Ant Design"
  echo "  -d, --dayjs      Install Day.js"
  echo "  -f, --faker      Install Faker"
  echo "  -i, --immer      Install Immer and use-immer"
  echo "  -m, --mui        Install MUI, styled-components and @mui/styled-engine-sc"
  echo "  -q, --query      Install @tanstack/react-query"
  echo "  --query-dev      Install @tanstack/react-query-devtools"
  echo "  -r, --router     Install react-router-dom"
  echo "  -s, --styled     Install styled-components"
  echo "  -t, --tw         Install TailwindCSS and configure"
  echo "  -x, --axios      Install Axios"
  echo "  --msw            Install and initialize MSW"
  echo "  -h, --help       Display this help and exit"
}

# Parse command-line options
while [[ "$#" -gt 0 ]]; do
  case "$1" in
    -T|--template) template="$2"; shift ;;
    -N) projectName="$2"; shift ;;
    -P|--plugin) plugin="$2"; shift ;;
    -a|--antd) antd=true; packages_installed+=("Ant Design") ;;
    -d|--dayjs) dayjs=true; packages_installed+=("Day.js") ;;
    -f|--faker) faker=true; packages_installed+=("Faker") ;;
    -i|--immer) immer=true; packages_installed+=("Immer and use-immer") ;;
    -m|--mui) mui=true; packages_installed+=("MUI, styled-components, @mui/styled-engine-sc") ;;
    -q|--query) query=true; packages_installed+=("@tanstack/react-query") ;;
    --query-dev) query_dev=true; packages_installed+=("@tanstack/react-query-devtools") ;;
    -r|--router) router=true; packages_installed+=("react-router-dom") ;;
    -s|--styled) styled=true; packages_installed+=("styled-components") ;;
    -t|--tw) tw=true; packages_installed+=("TailwindCSS") ;;
    -x|--axios) axios=true; packages_installed+=("Axios") ;;
    --msw) msw=true; packages_installed+=("MSW") ;;
    -h|--help) show_help; exit 0 ;;
    *) echo "Unknown parameter passed: $1"; show_help; exit 1 ;;
  esac
  shift
done

# Check required parameters
if [ -z "$projectName" ]; then
  echo "Project name is required."
  show_help
  exit 1
fi

# Create project with Vite
npm create vite@latest $projectName -- --template $template

# Navigate into the project directory
cd $projectName

# Open in Visual Studio Code (if installed)
command -v code >/dev/null 2>&1 && code .

# Conditional installations
[ "$mui" = true ] && npm i @mui/material @mui/styled-engine-sc styled-components
[ "$router" = true ] && npm i react-router-dom
[ "$antd" = true ] && npm i antd
[ "$dayjs" = true ] && npm i dayjs
[ "$faker" = true ] && npm i -D @faker-js/faker
[ "$immer" = true ] && npm i immer use-immer
[ "$axios" = true ] && npm i axios
[ "$query" = true ] && npm i @tanstack/react-query
[ "$styled" = true ] && npm i styled-components
[ "$query_dev" = true ] && npm i @tanstack/react-query-devtools
[ "$tw" = true ] && {
  npm install -D tailwindcss postcss autoprefixer
  npx tailwindcss init -p
  # Configure tailwind.config.js
  echo "/** @type {import('tailwindcss').Config} */" > tailwind.config.js
  echo "export default {" >> tailwind.config.js
  echo "  content: [" >> tailwind.config.js
  echo "    './index.html'," >> tailwind.config.js
  echo "    './src/**/*.{js,ts,jsx,tsx}'," >> tailwind.config.js
  echo "  ]," >> tailwind.config.js
  echo "  theme: {" >> tailwind.config.js
  echo "    extend: {}," >> tailwind.config.js
  echo "  }," >> tailwind.config.js
  echo "  plugins: []," >> tailwind.config.js
  echo "}" >> tailwind.config.js
  # Prepend styles to index.css
  echo -e "@tailwind base;\n@tailwind components;\n@tailwind utilities;\n$(cat src/index.css)" > src/index.css
}
[ "$msw" = true ] && {
  npm i -D msw@latest
  npx msw init "./public"
}
[ "$plugin" = "svgr" ] && {
  npm i -D vite-plugin-svgr
  [[ -f "vite-env.d.ts" ]] && echo "/// <reference types=\"vite-plugin-svgr/client\" />" >> vite-env.d.ts
  echo -e "\e[33mPlease manually add SVGR plugin to 'vite.config.js'\e[0m"
}

# Initialize version control
git init

# List installed packages
echo "Installed packages:"
for pkg in "${packages_installed[@]}"; do
  echo "- $pkg"
done

echo "Setup complete."

```

