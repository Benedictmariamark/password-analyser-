# password-analyser-
# password_tool.py

import argparse
from zxcvbn import zxcvbn
import itertools

# ---------- Password Strength Analysis ----------

def analyze_password_strength(password):
    result = zxcvbn(password)
    score = result['score']  # 0 (weak) to 4 (strong)
    suggestions = result['feedback']['suggestions']
    
    print(f"\n[+] Password Strength Score (0-4): {score}")
    if suggestions:
        print("[!] Suggestions:")
        for s in suggestions:
            print(f"  - {s}")
    else:
        print("✅ No suggestions – strong password!")

# ---------- Leetspeak Variants Generator ----------

def leetspeak_variants(word):
    leet_map = {'a': ['@', '4'], 'e': ['3'], 'i': ['1', '!'], 'o': ['0'], 's': ['$', '5'], 't': ['7']}
    variants = set([word])
    for i in range(len(word)):
        char = word[i].lower()
        if char in leet_map:
            for replacement in leet_map[char]:
                new_word = word[:i] + replacement + word[i+1:]
                variants.add(new_word)
    return list(variants)

# ---------- Custom Wordlist Generator ----------

def generate_wordlist(keywords, years=range(1990, 2030)):
    patterns = set()
    for word in keywords:
        variants = leetspeak_variants(word)
        for var in variants:
            patterns.add(var)
            for year in years:
                patterns.add(f"{var}{year}")
                patterns.add(f"{year}{var}")
                patterns.add(f"{var}@{year}")
    return list(patterns)

# ---------- Export Wordlist to .txt ----------

def export_wordlist(wordlist, filename="custom_wordlist.txt"):
    with open(filename, "w") as f:
        for word in wordlist:
            f.write(word + "\n")
    print(f"\n[+] Wordlist exported to: {filename}")

# ---------- Main CLI ----------

def main():
    parser = argparse.ArgumentParser(description="🔐 Password Analyzer & Wordlist Generator")
    parser.add_argument('-p', '--password', type=str, help='Password to analyze')
    parser.add_argument('-k', '--keywords', nargs='+', help='Custom keywords (e.g. name, pet, dob)')
    parser.add_argument('-o', '--output', type=str, default="custom_wordlist.txt", help='Output wordlist filename')
    args = parser.parse_args()

    if args.password:
        analyze_password_strength(args.password)

    if args.keywords:
        wordlist = generate_wordlist(args.keywords)
        export_wordlist(wordlist, args.output)

if __name__ == "__main__":
    main()

# password_gui_tool.py

import tkinter as tk
from tkinter import messagebox, filedialog
from zxcvbn import zxcvbn
import os

# ---------- Leetspeak Variants ----------
def leetspeak_variants(word):
    leet_map = {'a': ['@', '4'], 'e': ['3'], 'i': ['1', '!'], 'o': ['0'], 's': ['$', '5'], 't': ['7']}
    variants = set([word])
    for i in range(len(word)):
        char = word[i].lower()
        if char in leet_map:
            for replacement in leet_map[char]:
                new_word = word[:i] + replacement + word[i+1:]
                variants.add(new_word)
    return list(variants)

# ---------- Wordlist Generator ----------
def generate_wordlist(keywords, years=range(1990, 2030)):
    patterns = set()
    for word in keywords:
        variants = leetspeak_variants(word)
        for var in variants:
            patterns.add(var)
            for year in years:
                patterns.add(f"{var}{year}")
                patterns.add(f"{year}{var}")
                patterns.add(f"{var}@{year}")
    return sorted(patterns)

# ---------- GUI Functions ----------
def analyze_password():
    password = entry_password.get()
    if not password:
        messagebox.showwarning("Input Error", "Please enter a password to analyze.")
        return
    result = zxcvbn(password)
    score = result['score']
    suggestions = result['feedback']['suggestions']
    feedback = f"Strength Score (0-4): {score}\n"
    if suggestions:
        feedback += "\nSuggestions:\n" + "\n".join(f"- {s}" for s in suggestions)
    else:
        feedback += "\nStrong password!"
    text_output.delete("1.0", tk.END)
    text_output.insert(tk.END, feedback)

def generate_and_save_wordlist():
    raw_keywords = entry_keywords.get()
    if not raw_keywords.strip():
        messagebox.showwarning("Input Error", "Enter at least one keyword.")
        return
    keywords = raw_keywords.split()
    wordlist = generate_wordlist(keywords)
    filepath = filedialog.asksaveasfilename(defaultextension=".txt", title="Save Wordlist", filetypes=[("Text Files", "*.txt")])
    if filepath:
        with open(filepath, "w") as f:
            for word in wordlist:
                f.write(word + "\n")
        messagebox.showinfo("Success", f"Wordlist saved to:\n{filepath}")

# ---------- GUI Layout ----------
root = tk.Tk()
root.title("🔐 Password Analyzer & Wordlist Generator")
root.geometry("500x500")
root.resizable(False, False)

tk.Label(root, text="Enter Password to Analyze:", font=('Arial', 12)).pack(pady=5)
entry_password = tk.Entry(root, show="*", font=('Arial', 12), width=40)
entry_password.pack()

tk.Button(root, text="Analyze Password", command=analyze_password, font=('Arial', 11), bg="#007acc", fg="white").pack(pady=10)

tk.Label(root, text="Enter Keywords (space-separated):", font=('Arial', 12)).pack(pady=5)
entry_keywords = tk.Entry(root, font=('Arial', 12), width=40)
entry_keywords.pack()

tk.Button(root, text="Generate and Save Wordlist", command=generate_and_save_wordlist, font=('Arial', 11), bg="#28a745", fg="white").pack(pady=10)

tk.Label(root, text="Output:", font=('Arial', 12)).pack()
text_output = tk.Text(root, height=10, width=60, font=('Consolas', 10))
text_output.pack(pady=5)

root.mainloop()

