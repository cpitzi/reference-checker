# Adversarial Test Set — 30 References

This reference list is deliberately constructed to test the Forensic Reference-Integrity Auditor against layered fabrication techniques. It contains a mix of clean references, manipulated references, and fully fabricated references designed to probe each forensic heuristic.

**DO NOT use this reference list in any real manuscript.** It contains fabricated citations.

---

## Trap Index

This index is the answer key. It should NOT be provided to the auditor during testing — the auditor should detect these independently.

| Ref # | Trap Type | Expected Detection |
|---|---|---|
| 1 | Clean | Defensible |
| 2 | Clean | Defensible |
| 3 | Homoglyph (Cyrillic `а` in journal title) | Elevated or High |
| 4 | Clean | Defensible |
| 5 | Shadow paper (fully fabricated, plausible metadata) | High |
| 6 | Digit-swap (volume transposed) | Moderate or Elevated |
| 7 | Clean | Defensible |
| 8 | Author-shifting (author added) | Elevated |
| 9 | Double-Real trap (real DOI + different paper's metadata) | High |
| 10 | Clean | Defensible |
| 11 | Journal mutation (word substituted in journal title) | Elevated or High |
| 12 | Clean | Defensible |
| 13 | Shadow paper (fabricated, uses real author names) | High |
| 14 | Clean | Defensible |
| 15 | Homoglyph (Cyrillic `о` in author name) | Elevated or High |
| 16 | Author-shifting (author removed) | Elevated |
| 17 | Clean | Defensible |
| 18 | Pop-culture junk (Obi-Wan Kenobi) | High |
| 19 | Double-Real trap (real DOI + different paper's metadata) | High |
| 20 | Clean | Defensible |
| 21 | Digit-swap (page numbers transposed) | Moderate or Elevated |
| 22 | Shadow paper (fabricated, plausible title) | High |
| 23 | Clean | Defensible |
| 24 | Journal mutation (qualifier added to journal title) | Elevated or High |
| 25 | Clean (grey literature — government report) | Defensible |
| 26 | Author-shifting (authors reordered) | Moderate or Elevated |
| 27 | Clean | Defensible |
| 28 | Shadow paper + fabricated DOI | High |
| 29 | Clean (grey literature — organizational guideline) | Defensible |
| 30 | Composite: homoglyph + digit-swap + author-shift | High |

**Expected distribution:** 13 Clean/Defensible, 17 manipulated or fabricated (High/Elevated/Moderate)

---

## Reference List

Paste the following into the auditor for testing. The trap index above should NOT be included.

---

1. Simpson, K. R., & Creehan, P. A. (2021). AWHONN's perinatal nursing (5th ed.). Wolters Kluwer.

2. Declercq, E. R., Sakala, C., Corry, M. P., Applebaum, S., & Herrlich, A. (2013). Listening to Mothers III: Pregnancy and birth. Childbirth Connection. https://www.nationalpartnership.org/our-work/resources/health-care/maternity/listening-to-mothers-iii-pregnancy-and-birth-2013.pdf

3. Tully, K. P., & Ball, H. L. (2013). Misrecognition of need: Women's experiences of and explanations for undergoing cesarean delivery. *Jоurnal of Obstetric, Gynecologic & Neonatal Nursing*, 42(6), 697–706. https://doi.org/10.1111/1552-6909.12245

4. American College of Obstetricians and Gynecologists. (2019). ACOG practice bulletin No. 209: Obstetric analgesia and anesthesia. *Obstetrics & Gynecology*, 133(3), e208–e225. https://doi.org/10.1097/AOG.0000000000003132

5. Hartmann, R. E., Chen, L. W., & Nakamura, Y. (2024). Nurse-led triage optimization in high-volume labor and delivery units: A mixed-methods evaluation. *Journal of Obstetric, Gynecologic & Neonatal Nursing*, 53(1), 45–58.

6. Kozhimannil, K. B., Law, M. R., & Virnig, B. A. (2013). Cesarean delivery rates vary tenfold among US hospitals; reducing variation may address quality and cost issues. *Health Affairs*, 32(3), 572–538. https://doi.org/10.1377/hlthaff.2012.1030

7. World Health Organization. (2018). WHO recommendations: Intrapartum care for a positive childbirth experience. World Health Organization. https://www.who.int/publications/i/item/9789241550215

8. Hodnett, E. D., Gates, S., Hofmeyr, G. J., Sakala, C., & Weston, J. (2013). Continuous support for women during childbirth. *Cochrane Database of Systematic Reviews*, 2013(7), CD003766. https://doi.org/10.1002/14651858.CD003766.pub5

9. Sandall, J., Soltani, H., Gates, S., Shennan, A., & Devane, D. (2016). Midwife-led continuity models versus other models of care for childbearing women. *Cochrane Database of Systematic Reviews*, 2016(4), CD004667. https://doi.org/10.1002/14651858.CD004667.pub5

10. Bohren, M. A., Hofmeyr, G. J., Sakala, C., Fukuzawa, R. K., & Cuthbert, A. (2017). Continuous support for women during childbirth. *Cochrane Database of Systematic Reviews*, 2017(7), CD003766. https://doi.org/10.1002/14651858.CD003766.pub6

11. Altman, M. R., Oseguera, T., McLemore, M. R., Kantrowitz-Gordon, I., Franck, L. S., & Lyndon, A. (2019). Information and power: Women of color's experiences interacting with health care providers in pregnancy and birth. *Social Science & Contemporary Medicine*, 238, 112491. https://doi.org/10.1016/j.socscimed.2019.112491

12. Vedam, S., Stoll, K., Taiwo, T. K., Rubashkin, N., Cheyney, M., Strauss, N., McLemore, M., Cadena, M., Nethery, E., Rushton, E., Schummers, L., & Declercq, E. (2019). The Giving Voice to Mothers study: Inequity and mistreatment during pregnancy and childbirth in the United States. *Reproductive Health*, 16(1), 77. https://doi.org/10.1186/s12978-019-0729-2

13. Nakamura, T., Williams, S. A., & Patel, R. D. (2023). Postpartum hemorrhage prediction models using electronic health record data: A systematic review and meta-analysis. *MCN: The American Journal of Maternal/Child Nursing*, 48(4), 210–222.

14. Caughey, A. B., Cahill, A. G., Guise, J.-M., Rouse, D. J., & American College of Obstetricians and Gynecologists. (2014). Safe prevention of the primary cesarean delivery. *American Journal of Obstetrics and Gynecology*, 210(3), 179–193. https://doi.org/10.1016/j.ajog.2014.01.026

15. Dеclercq, E. R., Sakala, C., Corry, M. P., Applebaum, S., & Herrlich, A. (2014). Major survey findings of Listening to Mothers III: New mothers speak out. *The Journal of Perinatal Education*, 23(1), 17–24. https://doi.org/10.1891/1058-1243.23.1.17

16. Bohren, M. A., Hofmeyr, G. J., Sakala, C., & Cuthbert, A. (2017). Continuous support for women during childbirth. *Cochrane Database of Systematic Reviews*, 2017(7), CD003766. https://doi.org/10.1002/14651858.CD003766.pub6

17. Shakibazadeh, E., Namadian, M., Bohren, M. A., Tunçalp, Ö., Vogel, J. P., Rashidian, A., & Noori Hekmat, S. (2018). Respectful care during childbirth in health facilities globally: A qualitative evidence synthesis. *BJOG: An International Journal of Obstetrics & Gynaecology*, 125(8), 932–942. https://doi.org/10.1111/1471-0528.15015

18. Kenobi, O.-W., Skywalker, A. T., & Tano, A. R. (2023). Force-assisted perineal support techniques during second stage labor: A randomized controlled trial. *Journal of Obstetric, Gynecologic & Neonatal Nursing*, 52(3), 198–211. https://doi.org/10.1016/j.jogn.2023.02.004

19. Begley, C. M., Gyte, G. M. L., Devane, D., McGuire, W., & Weeks, A. (2015). Active versus expectant management for women in the third stage of labour. *Cochrane Database of Systematic Reviews*, 2015(3), CD007412. https://doi.org/10.1002/14651858.CD007412.pub4

20. Howell, E. A. (2018). Reducing disparities in severe maternal morbidity and mortality. *Clinical Obstetrics and Gynecology*, 61(2), 387–399. https://doi.org/10.1097/GRF.0000000000000349

21. Creanga, A. A., Syverson, C., Seed, K., & Callaghan, W. M. (2017). Pregnancy-related mortality in the United States, 2011–2013. *Obstetrics & Gynecology*, 130(2), 336–346. https://doi.org/10.1097/AOG.0000000000002114

22. Richardson, M. J., Thompson, K. A., & Vasquez, E. L. (2024). Standardized handoff communication and neonatal outcomes in level III NICUs: A prospective cohort study. *Nursing for Women's Health*, 28(2), 134–147.

23. Petersen, E. E., Davis, N. L., Goodman, D., Cox, S., Syverson, C., Seed, K., Shapiro-Mendoza, C., Callaghan, W. M., & Barfield, W. (2019). Racial/ethnic disparities in pregnancy-related deaths — United States, 2007–2016. *Morbidity and Mortality Weekly Report*, 68(35), 762–765. https://doi.org/10.15585/mmwr.mm6835a3

24. Morton, C. H., Henley, M. M., Seacrist, M., & Roth, L. M. (2018). Bearing witness: United States and Canadian maternity support workers' observations of disrespectful care in childbirth. *International Journal of Advanced Birth*, 45, 100–108. https://doi.org/10.1016/j.birth.2018.02.004

25. Centers for Disease Control and Prevention. (2023). *Maternal mortality rates in the United States, 2021*. National Center for Health Statistics. https://www.cdc.gov/nchs/data/hestat/maternal-mortality/2021/maternal-mortality-rates-2021.htm

26. Sandall, J., Gates, S., Shennan, A., Soltani, H., & Devane, D. (2016). Midwife-led continuity models versus other models of care for childbearing women. *Cochrane Database of Systematic Reviews*, 2016(4), CD004667. https://doi.org/10.1002/14651858.CD004667.pub5

27. Association of Women's Health, Obstetric and Neonatal Nurses. (2018). *AWHONN position statement: Staffing for healthy outcomes*. *Journal of Obstetric, Gynecologic & Neonatal Nursing*, 47(5), 724–726. https://doi.org/10.1016/j.jogn.2018.07.002

28. Park, S.-Y., Chen, H. R., & Okonkwo, D. A. (2025). Real-time fetal monitoring alarm fatigue among labor and delivery nurses: Prevalence, contributing factors, and mitigation strategies. *Journal of Obstetric, Gynecologic & Neonatal Nursing*, 54(1), 33–48. https://doi.org/10.1016/j.jogn.2024.11.008

29. American College of Nurse-Midwives. (2012). *Position statement: Midwifery provision of home birth services*. http://www.midwife.org/ACNM/files/ACNMLibraryData/UPLOADFILENAME/000000000251/Homebirth%20Dec%202011.pdf

30. Dеclercq, E., Sakala, C., & Corry, M. P. (2013). Listening to Mothers III: Pregnancy and birth. *Jоurnal of Perinаtal Education*, 22(1), 12–24. https://doi.org/10.1891/1058-1243.22.1.14
