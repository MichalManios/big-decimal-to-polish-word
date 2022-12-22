# big-decimal-to-polish-word
Zamiana BigDecimal na Stringa - na kwotę słownie po Polsku.



public String inWords(BigDecimal valueToText) {

        String[] unitsWords = {"", "jeden ", "dwa ", "trzy ", "cztery ", "pięć ",
            "sześć ", "siedem ", "osiem ", "dziewięć "};

        String[] teensWords = {"", "jedenaście ", "dwanaście ", "trzynaście ", "czternaście ", "piętnaście ",
            "szesnaście ", "siedemnaście ", "osiemnaście ", "dziewiętnaście "};

        String[] dozensWords = {"", "dziesięć ", "dwadzieścia ", "trzydzieści ", "czterdzieści ", "pięćdziesiąt ",
            "sześćdziesiąt ", "siedemdziesiąt ", "osiemdziesiąt ", "dziewięćdziesiąt "};

        String[] hundredsWords = {"", "sto ", "dwieście ", "trzysta ", "czterysta ", "pięćset ",
            "sześćset ", "siedemset ", "osiemset ", "dziewięćset "};

        String[][] groupsWords = {{"", "", ""},
            {"tysiąc ", "tysiące ", "tysięcy "},
            {"milion ", "miliony ", "milionów "},
            {"miliard ", "miliardy ", "miliardów "},
            {"bilion ", "biliony ", "bilionów "},
            {"biliard ", "biliardy ", "biliardów "},
            {"trylion ", "tryliony ", "trylionów "}};

        int units, teens, dozens, hundreds, groups = 0 , ends;

        StringBuilder inWords = new StringBuilder();
        String mark = "";

        BigDecimal thousand = new BigDecimal(1000);
        BigDecimal hundred = new BigDecimal(100);
        BigDecimal ten = new BigDecimal(10);

        MathContext precision = new MathContext(0);

        String afterTheDecimal = valueToText.remainder(BigDecimal.ONE).movePointRight(valueToText.scale())
            .abs().toString();

        if (valueToText.compareTo(BigDecimal.ZERO) < 0) {
            mark = "minus ";
            valueToText = valueToText.negate();
        }

        if (valueToText.compareTo(BigDecimal.ZERO) == 0) {
            mark = "zero ";
        }

        while (valueToText.intValue() != 0) {
            hundreds = valueToText.remainder(thousand).intValue() / 100;
            dozens = valueToText.remainder(hundred).intValue() / 10;
            units = valueToText.remainder(ten).intValue();

            if (dozens == 1 && units > 0) {
                teens = units;
                dozens = 0;
                units = 0;
            } else {
                teens = 0;
            }

            if (units == 1 && hundreds + dozens + teens == 0) {
                ends = 0;

                if (hundreds + dozens == 0 && groups > 0) {
                    units = 0;
                    inWords.insert(0, groupsWords[groups][ends]);
                }

            } else {
                switch (units) {
                    case 2, 3, 4 -> ends = 1;
                    default -> ends = 2;
                }
            }

            if (hundreds + dozens + teens + units > 0) {
                inWords.insert(0, hundredsWords[hundreds]
                    + dozensWords[dozens]
                    + teensWords[teens]
                    + unitsWords[units]
                    + groupsWords[groups][ends]);
            }

            valueToText = valueToText.divide(thousand, precision);
            groups = groups + 1;
        }

        inWords = new StringBuilder(mark + inWords + "zł " + afterTheDecimal + "/100");
        return inWords.toString();
    }
