# Cancer screening Care Algorithm

## Rationale

Appropriate cancer screening is an important part of effective preventive health management. Inappropriate cancer screening is a cause of medical harm. Lack of shared decision making contributes to inappropriate cancer screening that is not consistent with a patients preferences. Failure to followup on cancer screening results is a leading cause of malpractice claims against internists and primary care providers (ref CRICO report).

An example list of cancer screening is:

```txt
Breast[family history, F>40]: no significant family history|family history of ***, does not apply | discussed, defer | prefers annual | prefers biannual | prefers no screening, next review ***
Cervical[F 21-29 q3y, F>=30 PAP+HPV q5y]: history of abnormal pap ***, next pap ***
Colon[>=50 discuss, <50 +RF]: risk factors ****, prefers ***, next ****
Lung[55-74 >=30 pack year within 15 years]: non smoker | less than 30 pack years | quit > 15 years ago | declined | screened on ***
Prostate[>=50, <50 +RF(fam hx, aa)]: discussed, declines screening | annual, next ***
Melanoma[annual derm if RF(fam history, fair skin, sun burns)]: ***
```

## DSL

```ruby
define "Breast cancer screening" do

  estimate inherited_risk do
    if ethnicity is Jewish
      high risk if any
        one or more cases of breast cancer <= 50 years old in first or second degree relative
        one or more cases of ovarian cancer any age in first or second degree relative
        one or more cases of first or second degree relative with breast or ovarian cancer any age any gender
        any high risk genetics for breast cancer (such as BRCA1 or BRCA2)
    else #not Jewish
      high risk if any
        one or more cases of breast cancer in first or second degree relative <= 40 years old
        one or more cases of first or second degree relative with breast or ovarian cancer any age any gender
        2 or more cases of breast cancer if (diagnosed <= 50 or bilateral cancer)
        any high risk genetics for breast cancer (such as BRCA1 or BRCA2)
    end
  end

  estimate female_breast_cancer_risk_category do

    high risk with RR 3.0 if has "BRCA1 or BRCA2 genes"
    high risk with RR 2.6 if has mother or sister with breast cancer
    case age
      when 30-34
        low risk
      when 70-74
         high risk with RR 18
    end
    case age at "menarche"
      when >14
        low risk
      when <12
        high risk with RR 1.9
    end
    case age at "menopause"
      when <45
        low risk
      when >55
        high risk with RR 2.0
    end
    case has_used_or_is_using contraceptive_pills
      when never
        low risk
      when past or current
        high risk with RR 1.07
    end
    case is_using hormone_replacement_therapy
      when never
        low risk
      when current
        high risk with RR 1.2
    end

    case drinks alcohol
      when 0
        low risk
      when >=2
        high risk with RR 1.4
    end

    case breast_density_percent
      when 0
        low risk
      when >=75
        high risk with RR 1.8
    end

    case bone_density_quartile
      when 1
        low risk
      when 4
        high risk with RR 1.7
    end

    high risk with RR 1.7 if history_of breast_biopsy == benign
    high risk with RR 3.7 if history_of breast_biopsy == atypical_hyperplasia

    # protective factors
    low risk with RR 0.73 if history_of breast_feeding_months >= 16
    low risk with RR 0.71 if parity >= 5
    low risk with RR 0.70 if exercises?
    if is_postmenopause
      case BMI
        when <22.9
          low risk with RR 0.63
        when >30.7
          high risk
      end
    end

    low risk with RR 0.3 if history_of oophorectomy? && age_when_had oopherectomy <35

    low risk if takes aspirin at least per week for >= 6 months
  end

  def shared_decision_making(risk_category)
    inform of risk category and any unknowns
    show possible interventions with risk/benefit
    get choice
  end

  def recommendation
    unknowns = [] # keeps track of any unknown answers

    exclude is_female? and is_pregnant?, "These recommendations are for non pregnant women only."
    exclude has breast cancer
    exclude has_had bilateral mastectomy
    exclude age<18
  
    # these apply to males or females
    if estimate inherited_risk is high risk
      recommended_screening referral to genetics because of "high risk of inherited breast-ovarian cancer syndrome"
      return
    end

    if ask_or_get(personal_history_of breast_cancer) is positive?
        recommended_screening annual because "of a personal history of breast cancer" with strength "A"
        return
    end

    if is male?
        recommended_screening none because "low risk of breast cancer in males without gynecomastia or breast complaints"
    end

    risk_category = estimate female_breast_cancer_risk_category

    if no previous shared decision making or previous choice was defered then
      preference = shared decision making with risk_category
    elsif risk_category has increased from prior or patient wishes to readdress or has been greater than 5 years since was addressed then
      preference = shared decision making with risk_category
    end

    case preference
      when no screening
        recommended_screening none because "of patient choice after shared decision making"
      when screening
        # wants whatever is recommended for her risk category
        case risk_category
          when high risk
            recommended_screening annual because "high risk"
          else
            recommended_screening biannual because "average or low risk"
        end
      when annual screening
        recommended_screening annual because "of patient preference after shared decision making"
      when biannual screening
        recommended_screening biannual because "of patient preference after shared decision making"
      when defer
        recommended_screening defer because "of patient preference after shared decision making"
    end
  end

end
```

## User Scenarios

* A user is scheduled to see for an annual wellness visit.
* Her care algorithms are continuously run, when active. She does not have active problems. Her preventive algorithms are run if never run before, or at a scheduled time a few times a year
* The algorithm for breast cancer screening is run
* she does not yet have a recommended screening (or it could have expired)
* the DSL has been scanned, and needed data elements have been identified (in earlier versions of the DSL these were declared up front with depends on clauses)
  * some of the data depends on input from the patient (family history, breast risk category questions)
  * a conversation template is created for these questions and sent to the patient
  * answers from the conversation are captured in structured fields
* She is asked if she would like to do the shared decision making, or wait for the visit
* She chooses to do it independently
* She sees the results, and makes a selection, as she is 40 and low risk she chooses no screening.
* She has a link where she can redo the decision making at any time, or she will be prompted again in 5 years
* When she goes to her annual physical, her physician sees that she just made her selection about this (in their active document of her plan). The doctor asks if she has any questions, which she does not and confirms her choice. The confirmation is recorded associated with the decision.




