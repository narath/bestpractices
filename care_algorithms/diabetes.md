# Diabetes Type 2

## Rationale

Diabetes is one of the major chronic conditions worldwide. It results in significant morbidity related to heart disease, blindness, amputations and renal disease.

## Algorithm

```ruby

# all data should have provenance and date reported
data_definition do
  patient name
  date of birth
  primary insurance
  secondary insurance
  height
  weight
  bmi do
    height in meters ** 2 / weight in kg
  end
  bmi classification do
    if ethnicity == asian
      case bmi
        when >=27
          obese
        when >=23
          overweight
      end
    else
      case bmi
        when >= 30
          obese
        when >= 25
          overweight
      end
  end
  has hypertension?
  has cvd? = has any of (myocardial infarction, stroke, cabg, heart failure, hypertension, coronary artery disease, peripheral arterial disease)
  medications
  prior medications # why prescribed and why stopped would be a major breakthrough
  medication allergies
  gfr # glomerular filtration rate
  a1c
  history_of hypoglycemia?

  activities of daily living aka ADL do
    # todo 
  end

  instrumental activities of daily living aka IADL do
    # todo
  end

end


exclusion end stage medical condition
exclusion moderate to severe dementia
exclusion long term care resident
exclusion impaired adls >= 2

define_target
recommend diet and lifestyle changes

if not at target
  def atypical?
    (bmi == normal && has_no hypertension? && has_no family_history_of diabetes) ||
    (unintentional weight loss > 10 pounds)
  end
  refer_to endocrine if atypical?

  medication review and discontinue any medications that could could contribute to diabetes (thiazides)

   recommend metformin do
    no allergy to metformin
    no contraindications to metformin
    has recent GFR (in last 6 months)
    max dose do
      GFR < 30: contraindicated
      GFR < 45: 1000mg
      else: 2000mg
    end
    recommend order metformin to physician (with weekly increase to maximum tolerated dose) ->
      on(accept) 
        recommend pickup metformin to patient ->
          on(accept) 
            send instructions on metformin, how to take, side effects
            send starting new medication conversation with patient, metformin
            schedule education and side effect monitoring in 3 days
            schedule 3 month A1C check (reminder to order A1c 1 week prior -> reminder to patient -> confirm lab result -> interpret)
          end
          on(reject, reason)
            escalate "metformin rejected by patient with ", reason
          end
      end
      on(reject, reason)
        if reason = "could not tolerate" or "other medication preferred"
          second line medication
        end
    end
end


```


## References


