# big-decimal-to-polish-word
Zamiana BigDecimal na Stringa - na kwotę słownie po Polsku z nazwą waluty odmienianą w zależności od końcówki.

    public String inWords(BigDecimal value) {
        BigDecimal valueToText = value;
        return convertNumberToWords(valueToText);
    }
    
    private String convertNumberToWords(BigDecimal valueToText){

        String[] unitsWords = {"", "jeden ", "dwa ", "trzy ", "cztery ", "pięć ",
            "sześć ", "siedem ", "osiem ", "dziewięć "};

        String[] teensWords = {"", "jedenaście ", "dwanaście ", "trzynaście ", "czternaście ", "piętnaście ",
            "szesnaście ", "siedemnaście ", "osiemnaście ", "dziewiętnaście "};

        String[] dozensWords = {"", "dziesięć ", "dwadzieścia ", "trzydzieści ", "czterdzieści ", "pięćdziesiąt ",
            "sześćdziesiąt ", "siedemdziesiąt ", "osiemdziesiąt ", "dziewięćdziesiąt "};

        String[] hundredsWords = {"", "sto ", "dwieście ", "trzysta ", "czterysta ", "pięćset ",
            "sześćset ", "siedemset ", "osiemset ", "dziewięćset "};

        String[] currencyVariation = {"złoty ", "złote ", "złotych "};

        String[][] groupsWords = {{"", "", ""},
            {"tysiąc ", "tysiące ", "tysięcy "},
            {"milion ", "miliony ", "milionów "},
            {"miliard ", "miliardy ", "miliardów "},
            {"bilion ", "biliony ", "bilionów "},
            {"biliard ", "biliardy ", "biliardów "},
            {"trylion ", "tryliony ", "trylionów "}};

        int units, teens, dozens, hundreds, groups = 0, ends = 0, unityOfTheAmount;

        StringBuilder inWords = new StringBuilder();
        String mark = "";

        BigDecimal thousand = new BigDecimal(1000);
        BigDecimal hundred = new BigDecimal(100);
        BigDecimal ten = new BigDecimal(10);

        MathContext precision = new MathContext(0);

        String afterTheDecimal = valueToText.remainder(BigDecimal.ONE).movePointRight(valueToText.scale())
            .abs().toString();

        mark = checkIfZero(valueToText, mark);
        mark = checkIfNegative(valueToText, mark);
        valueToText = checkIfNeedChangeToPositive(valueToText);

        unityOfTheAmount = valueToText.remainder(ten).intValue();

        while (valueToText.intValue() != 0) {
            hundreds = countHundreds(valueToText, thousand);
            dozens = countDozens(valueToText, hundred);
            units = valueToText.remainder(ten).intValue();

            if (dozens == 1 && units > 0) {
                teens = units;
                dozens = 0;
                units = 0;
            } else {
                teens = 0;
            }

            ends = calculateEnds(units, hundreds, dozens, teens);

            addWordsToTheString(hundredsWords, dozensWords, teensWords, unitsWords, groupsWords,
                groups, ends, hundreds, dozens, teens, units, inWords);

            valueToText = divideByThousand(valueToText, precision, thousand);
            groups = increaseGroup(groups);
        }

        ends = calculateEndsForCurrency(groups, unityOfTheAmount, ends, mark);

        inWords = addCurrencyVariationToString(currencyVariation, ends, inWords, afterTheDecimal);

        if (inWords.charAt(0) == ' '){
            inWords.deleteCharAt(0);
        }

        return mark + inWords;
    }

    private BigDecimal checkIfNeedChangeToPositive(BigDecimal valueToText) {
        if (valueToText.compareTo(BigDecimal.ZERO) < 0) {
            valueToText = valueToText.negate();
        }
        return valueToText;
    }

    private String checkIfNegative(BigDecimal valueToText, String mark) {
        if (valueToText.compareTo(BigDecimal.ZERO) < 0) {
            mark = "minus ";
        }
        return mark;
    }

    private String checkIfZero(BigDecimal valueToText, String mark) {
        if (valueToText.compareTo(BigDecimal.ZERO) == 0) {
            mark = "zero ";
        }
        return mark;
    }

    private int countHundreds(BigDecimal valueToText, BigDecimal thousand) {
        return valueToText.remainder(thousand).intValue() / 100;
    }

    private int countDozens(BigDecimal valueToText, BigDecimal hundred) {
        return valueToText.remainder(hundred).intValue() / 10;
    }

    private int calculateEnds(int units, int hundreds, int dozens, int teens) {
        int ends;
        if (units == 1 && hundreds + dozens + teens == 0) {
            ends = 0;
        } else {
            switch (units) {
                case 2, 3, 4 -> ends = 1;
                default -> ends = 2;
            }
        }
        return ends;
    }

    private void addWordsToTheString(String[] hundredsWords, String[] dozensWords,
                                     String[] teensWords, String[] unitsWords, String[][] groupsWords, int groups, int ends,
                                     int hundreds, int dozens, int teens, int units, StringBuilder inWords) {
        if (hundreds + dozens == 0 && groups > 0) {
            inWords.insert(0, unitsWords[units] + groupsWords[groups][ends]);
        } else if (hundreds + dozens + teens + units > 0 || groups == 0) {
            inWords.insert(0, hundredsWords[hundreds]
                + dozensWords[dozens]
                + teensWords[teens]
                + unitsWords[units]
                + groupsWords[groups][ends]);
        }
    }

    private BigDecimal divideByThousand(BigDecimal valueToText, MathContext precision, BigDecimal thousand) {
        return valueToText.divide(thousand, precision);
    }

    private int increaseGroup(int groups) {
        return groups + 1;
    }

    private int calculateEndsForCurrency(int groups, int unityOfTheAmount, int ends, String mark) {
        if ((groups - 1 > 0 && (unityOfTheAmount < 2 || unityOfTheAmount > 4)) || mark.equals("zero ")) {
            ends = 2;
        } else if (groups - 1 > 0) {
            ends = 1;
        }
        return ends;
    }

    private StringBuilder addCurrencyVariationToString(String[] currencyVariation, int ends,
                                                       StringBuilder inWords, String afterTheDecimal) {
        inWords = new StringBuilder(inWords + currencyVariation[ends] + afterTheDecimal + "/100");
        return inWords;
    }
