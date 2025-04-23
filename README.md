# specifications
# CF-Compliant NetCDF Specification Guide

This document outlines the standards and conventions for creating [Climate and Forecast (CF)](http://cfconventions.org/) compliant NetCDF files for atmospheric and environmental data. The CF conventions aim to promote the processing and sharing of climate and forecast data using self-describing, portable NetCDF files.

---

## 📦 File Structure

A CF-compliant NetCDF file typically includes:

- **Dimensions** – named axes of the data (e.g., time, latitude, longitude, level).
- **Variables** – data arrays, each associated with a subset of the dimensions.
- **Attributes** – metadata for the file or individual variables (units, standard names, etc.).

---

## ✅ Required Global Attributes

```text
title               : Brief description of the dataset
institution         : Name of the institution that created the data
source              : How the data was generated (e.g., model name, observational platform)
history             : Audit trail of modifications (auto-generated)
references          : Citations or documentation
comment             : Additional information about the dataset
Conventions         : "CF-1.10"  # or the applicable version
