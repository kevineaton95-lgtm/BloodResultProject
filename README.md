# BloodResultProject
A project to read and analyze blood results to bring clarity and information to a lay person.
Many people don't understand the values given on their blood tests. This project can analyze inpuf from a user and analyze according to normal ranges for age and gender specific blood properties. 
This project can also be a support tools for healthcare practitoners.

# ==========================================================
# ADVANCED BLOOD RESULT ANALYZER
# ==========================================================
# FEATURES:
# - Gender-specific ranges
# - Age-specific ranges
# - Color coded output
# - Saves history to CSV
# - PMB logic
# - Symptom matching
# - ICD10 suggestions
# - Emergency triage severity
# - Error handling for invalid input
#
# DISCLAIMER:
# Educational/decision-support only.
# NOT a medical diagnosis system.
# ==========================================================

import csv
import os
from datetime import datetime

# ----------------------------------------------------------
# ANSI COLOR CODES
# ----------------------------------------------------------

RED = "\033[91m"
GREEN = "\033[92m"
YELLOW = "\033[93m"
CYAN = "\033[96m"
RESET = "\033[0m"

# ----------------------------------------------------------
# REFERENCE RANGES
# ----------------------------------------------------------

def get_reference_ranges(gender, age):

    ranges = {

        "hemoglobin": {
            "male": (13.5, 17.5),
            "female": (12.0, 15.5),
            "child": (11.0, 14.5)
        },

        "wbc": (4.0, 11.0),

        "platelets": (150, 450),

        "glucose": (3.9, 7.0),

        "creatinine": {
            "male": (64, 104),
            "female": (49, 90),
            "child": (20, 80)
        },

        "sodium": (135, 145),

        "potassium": (3.5, 5.2),

        "alt": (7, 45)
    }

    if age < 18:
        hb_range = ranges["hemoglobin"]["child"]
        creat_range = ranges["creatinine"]["child"]
    else:
        hb_range = ranges["hemoglobin"][gender]
        creat_range = ranges["creatinine"][gender]

    return {
        "hemoglobin": hb_range,
        "wbc": ranges["wbc"],
        "platelets": ranges["platelets"],
        "glucose": ranges["glucose"],
        "creatinine": creat_range,
        "sodium": ranges["sodium"],
        "potassium": ranges["potassium"],
        "alt": ranges["alt"]
    }

# ----------------------------------------------------------
# ANALYSIS ENGINE
# ----------------------------------------------------------

def analyze_results(results, symptoms, gender, age):

    ranges = get_reference_ranges(gender, age)

    findings = []
    diagnoses = []
    icd10 = []
    pmb_conditions = []
    severity = "LOW"

    # HEMOGLOBIN

    hb = results["hemoglobin"]

    if hb is not None:

        low, high = ranges["hemoglobin"]

        if hb < low:
            findings.append("Low hemoglobin")
            diagnoses.append("Possible anemia")
            icd10.append("D64.9 - Anemia, unspecified")

        elif hb > high:
            findings.append("High hemoglobin")
            diagnoses.append("Possible polycythemia")
            icd10.append("D75.1 - Secondary polycythemia")

    # WBC

    wbc = results["wbc"]

    if wbc is not None:

        low, high = ranges["wbc"]

        if wbc > high:
            findings.append("High white cell count")
            diagnoses.append("Possible infection")
            icd10.append("B99.9 - Infection, unspecified")

        elif wbc < low:
            findings.append("Low white cell count")
            diagnoses.append("Possible immune suppression")
            icd10.append("D72.819 - Leukopenia")

    # GLUCOSE

    glucose = results["glucose"]

    if glucose is not None:

        low, high = ranges["glucose"]

        if glucose > high:
            findings.append("High glucose")
            diagnoses.append("Possible diabetes mellitus")
            icd10.append("E11.9 - Type 2 diabetes mellitus")

            pmb_conditions.append(
                "Diabetes may qualify as a PMB chronic condition."
            )

        elif glucose < low:
            findings.append("Low glucose")
            diagnoses.append("Hypoglycemia")
            icd10.append("E16.2 - Hypoglycemia")

            severity = "HIGH"

    # CREATININE

    creat = results["creatinine"]

    if creat is not None:

        low, high = ranges["creatinine"]

        if creat > high:
            findings.append("Elevated creatinine")
            diagnoses.append("Possible kidney dysfunction")
            icd10.append("N17.9 - Acute kidney failure")

            pmb_conditions.append(
                "Kidney disease may qualify under PMB regulations."
            )

            severity = "HIGH"

    # POTASSIUM

    potassium = results["potassium"]

    if potassium is not None:

        low, high = ranges["potassium"]

        if potassium > high:
            findings.append("High potassium")
            diagnoses.append("Hyperkalemia")
            icd10.append("E87.5 - Hyperkalemia")

            severity = "EMERGENCY"

        elif potassium < low:
            findings.append("Low potassium")
            diagnoses.append("Hypokalemia")
            icd10.append("E87.6 - Hypokalemia")

    # ALT

    alt = results["alt"]

    if alt is not None:

        low, high = ranges["alt"]

        if alt > high:
            findings.append("Elevated liver enzymes")
            diagnoses.append("Possible liver inflammation")
            icd10.append("K76.9 - Liver disease")

    # TRIAGE

    if (
        "chest pain" in symptoms or
        "shortness of breath" in symptoms
    ):
        severity = "EMERGENCY"

    return {
        "findings": findings,
        "diagnoses": diagnoses,
        "icd10": icd10,
        "pmb": pmb_conditions,
        "severity": severity
    }

# ----------------------------------------------------------
# SAVE TO CSV
# ----------------------------------------------------------

def save_to_csv(patient_name, results, analysis):

    file_exists = os.path.isfile("blood_history.csv")

    with open("blood_history.csv", "a", newline="") as file:

        writer = csv.writer(file)

        if not file_exists:
            writer.writerow([
                "Date",
                "Patient",
                "Hemoglobin",
                "WBC",
                "Platelets",
                "Glucose",
                "Creatinine",
                "Sodium",
                "Potassium",
                "ALT",
                "Severity"
            ])

        writer.writerow([
            datetime.now(),
            patient_name,
            results["hemoglobin"],
            results["wbc"],
            results["platelets"],
            results["glucose"],
            results["creatinine"],
            results["sodium"],
            results["potassium"],
            results["alt"],
            analysis["severity"]
        ])

# ----------------------------------------------------------
# SAFE INPUT FUNCTIONS
# ----------------------------------------------------------

def get_float(prompt):

    while True:

        value = input(prompt)

        if value.strip() == "":
            return None

        try:
            return float(value)

        except ValueError:
            print(
                RED +
                "ERROR: Please enter numbers only."
                + RESET
            )

def get_age(prompt):

    while True:

        value = input(prompt)

        try:

            age = int(value)

            if age < 0 or age > 120:
                print(
                    RED +
                    "ERROR: Please enter a realistic age."
                    + RESET
                )

            else:
                return age

        except ValueError:
            print(
                RED +
                "ERROR: Age must be a whole number."
                + RESET
            )

def get_gender(prompt):

    while True:

        value = input(prompt).lower()

        if value in ["male", "female"]:
            return value

        else:
            print(
                RED +
                "ERROR: Enter 'male' or 'female'."
                + RESET
            )

# ----------------------------------------------------------
# USER INPUT
# ----------------------------------------------------------

print(CYAN + "\n=== ADVANCED BLOOD ANALYZER ===\n" + RESET)

patient_name = input("Patient name: ")

gender = get_gender("Gender (male/female): ")

age = get_age("Age: ")

symptoms = input(
    "Symptoms (comma separated): "
).lower().split(",")

symptoms = [s.strip() for s in symptoms]

results = {

    "hemoglobin": get_float("Hemoglobin: "),
    "wbc": get_float("WBC: "),
    "platelets": get_float("Platelets: "),
    "glucose": get_float("Glucose: "),
    "creatinine": get_float("Creatinine: "),
    "sodium": get_float("Sodium: "),
    "potassium": get_float("Potassium: "),
    "alt": get_float("ALT: ")
}

# ----------------------------------------------------------
# ANALYZE
# ----------------------------------------------------------

analysis = analyze_results(
    results,
    symptoms,
    gender,
    age
)

# ----------------------------------------------------------
# DISPLAY RESULTS
# ----------------------------------------------------------

print(CYAN + "\n========== RESULTS ==========\n" + RESET)

if analysis["findings"]:

    print(YELLOW + "Findings:" + RESET)

    for item in analysis["findings"]:
        print("- " + item)

    print("\n" + YELLOW + "Possible Diagnoses:" + RESET)

    for item in set(analysis["diagnoses"]):
        print("- " + item)

    print("\n" + YELLOW + "ICD10 Suggestions:" + RESET)

    for item in set(analysis["icd10"]):
        print("- " + item)

    print("\n" + YELLOW + "PMB Logic:" + RESET)

    for item in set(analysis["pmb"]):
        print("- " + item)

else:
    print(GREEN + "No major abnormalities detected." + RESET)

# ----------------------------------------------------------
# TRIAGE OUTPUT
# ----------------------------------------------------------

severity = analysis["severity"]

if severity == "EMERGENCY":
    print(RED + f"\nTRIAGE LEVEL: {severity}" + RESET)

elif severity == "HIGH":
    print(YELLOW + f"\nTRIAGE LEVEL: {severity}" + RESET)

else:
    print(GREEN + f"\nTRIAGE LEVEL: {severity}" + RESET)

# ----------------------------------------------------------
# SAVE DATA
# ----------------------------------------------------------

save_to_csv(patient_name, results, analysis)

print(CYAN + "\nData saved to blood_history.csv" + RESET)

print(
    RED +
    "\nDISCLAIMER: Educational and decision-support only."
    "\nConsult a qualified healthcare professional."
    + RESET
)
