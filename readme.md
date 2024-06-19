# Git-Diff Json schematics:

### What it does?

It takes output of gitdiff (present in variable `diff_input` then generates a schema from it. By modifying the schema, we can reverse this process, ultimately generating `git diff file` from `JSON file`.

### python Code: 

```python
import re
import json

# Sample diff input as a string
diff_input = """
diff --git a/app/dashboard/layout.tsx b/app/dashboard/layout.tsx
index db8ade6..b54d623 100644
--- a/app/dashboard/layout.tsx
+++ b/app/dashboard/layout.tsx
@@ -4,7 +4,6 @@ import "./../globals.css";
 import Navbar from "@/components/main/Navbar";
 import Footer from "@/components/main/Footer";
 import Dashboard from "./page";
-import StarsCanvas from "@/components/main/StarBackground";
 
 const inter = Inter({ subsets: ["latin"] });
 
@@ -23,7 +22,7 @@ export default function RootLayout({
       <body
         className={`${inter.className} bg-[#030014] overflow-y-scroll overflow-x-hidden`}
       >
-        <StarsCanvas />
+        {/* <StarsCanvas /> */}
         {/* <Navbar /> */}
         <Dashboard />
         {/* <Footer /> */}
diff --git a/app/dashboard/page.tsx b/app/dashboard/page.tsx
index 256cd0c..54450af 100644
--- a/app/dashboard/page.tsx
+++ b/app/dashboard/page.tsx
@@ -1,14 +1,7 @@
 export default function Dashboard() {
   return (
     <main className="h-full w-full text-white">
-      <div className="flex flex-col gap-20">
-        <div>Name</div>
-        <div>email</div>
-        <div>Other details like github stats, cp stats, proj, exp so on...</div>
-        {/*
-         * // TODO connect with @OpenEdu's prublic profile builder
-         */}
-      </div>
+      <div className="flex flex-col gap-20">Just a dashboard</div>
     </main>
   );
 }
"""

# Function to parse the git diff input
def parse_diff(diff):
    files = []
    current_file = None

    for line in diff.splitlines():
        if line.startswith('diff --git'):
            if current_file:
                files.append(current_file)
            parts = line.split(' ')
            current_file = {
                'file_path': parts[2][2:],
                'changes': []
            }
        elif line.startswith('index'):
            continue
        elif line.startswith('---') or line.startswith('+++'):
            continue
        elif line.startswith('@@'):
            change = {
                'start': int(re.search(r'\-(\d+)', line).group(1)),
                'lines': []
            }
            current_file['changes'].append(change)
        elif line.startswith('+') or line.startswith('-') or line.startswith(' '):
            change['lines'].append(line)
    
    if current_file:
        files.append(current_file)
    
    return files

# Parse the diff input
parsed_diff = parse_diff(diff_input)

# Convert parsed diff to JSON
json_output = json.dumps(parsed_diff, indent=2)

# Print JSON output
print(json_output)
```

### JSON output:

```json
[
  {
    "file_path": "app/dashboard/layout.tsx",
    "changes": [
      {
        "start": 4,
        "lines": [
          " import Navbar from \"@/components/main/Navbar\";",
          " import Footer from \"@/components/main/Footer\";",
          " import Dashboard from \"./page\";",
          "-import StarsCanvas from \"@/components/main/StarBackground\";",
          " ",
          " const inter = Inter({ subsets: [\"latin\"] });",
          " "
        ]
      },
      {
        "start": 23,
        "lines": [
          "       <body",
          "         className={`${inter.className} bg-[#030014] overflow-y-scroll overflow-x-hidden`}",
          "       >",
          "-        <StarsCanvas />",
          "+        {/* <StarsCanvas /> */}",
          "         {/* <Navbar /> */}",
          "         <Dashboard />",
          "         {/* <Footer /> */}"
        ]
      }
    ]
  },
  {
    "file_path": "app/dashboard/page.tsx",
    "changes": [
      {
        "start": 1,
        "lines": [
          " export default function Dashboard() {",
          "   return (",
          "     <main className=\"h-full w-full text-white\">",
          "-      <div className=\"flex flex-col gap-20\">",
          "-        <div>Name</div>",
          "-        <div>email</div>",
          "-        <div>Other details like github stats, cp stats, proj, exp so on...</div>",
          "-        {/*",
          "-         * // TODO connect with @OpenEdu's prublic profile builder",
          "-         */}",
          "-      </div>",
          "+      <div className=\"flex flex-col gap-20\">Just a dashboard</div>",
          "     </main>",
          "   );",
          " }"
        ]
      }
    ]
  }
]

=== Code Execution Successful ===
```
