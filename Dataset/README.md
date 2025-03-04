## 배경
국민대학교 경영대학원 AI빅데이터/디지털마케팅전공과 경영대학에서 ‘제1회 국민대학교 AI빅데이터 분석 경진대회’를 개최합니다.

이번 대회에서는 Total HR Service를 제공하는 (주)스카우트의 후원을 받아 유연한 노동시장으로의 변화 흐름에 맞추어,

구직자 입장에서는 자신의 이력과 경력에 맞춤화된 채용 공고를 추천받을 수 있고 구인기업 입장에서는 공고에 적합한 인재를 미리 선별하는 도구로 활용할 수 있도록 채용공고 추천 알고리즘 개발을 제안합니다.

이력서 등 구직자 관련 데이터와 채용 공고 관련 데이터, 그리고 지원 히스토리 데이터를 활용하여 구직자에게 맞춤화된 채용 공고를 자동으로 추천할 수 있는 알고리즘을 개발함에 따라

지원자는 적성에 맞는 채용 공고에 지원하여 직무 만족도를 높이고 구인기업은 직종에 맞는 핵심 인재를 선발할 수 있으리라 기대할 수 있습니다.

## 주제
이력서 맞춤형 채용 공고 추천 AI 모델 개발

## 설명
이력서, 채용 공고 및 지원 히스토리 데이터를 활용하여 구직자에게 맞춤화된 채용 공고를 자동으로 추천할 수 있는 추천시스템 알고리즘 개발

## 주최 / 주관
주최 : 국민대학교 경영대학원, LINC+ 사업단
주관 : 국민대학교 경영대학, 국민대학교 경영대학원 AI빅데이터전공/디지털마케팅전공
운영 : 데이콘
후원 : (주)스카우트

**apply_train.csv [파일]**
- 이력서가 채용 공고에 실제 지원한 관계 목록 (히스토리)
  
**이력서 관련 데이터 [파일]**
- ```resume.csv```
- ```resume_certificate.csv```
- ```resume_education.csv```
- ```resume_language.csv```
  
**채용공고 관련 데이터 [파일]**
- ```recruitment.csv```
- ```company.csv```
  
**sample_submission.csv [파일] - 제출 양식**
- ```resume_seq``` : 추천을 진행할 이력서 고유 번호
- ```recruitment_seq``` : 이력서에 대해 추천한 채용 공고 고유 번호
- ```resume.csv```에 존재하는 모든 resume_seq에 대해서 5개의 채용 공고를 추천해야 합니다.
- 해당 이력서에서 실제 지원이 이루어졌던 채용 공고는 추천하지 않습니다.
- 중복된 채용 공고를 추천하거나, 5개가 아닌 개수의 채용 공고를 추천하는 경우 제출이 불가능합니다.

※ 상세 데이터 명세는 [링크](https://docs.google.com/spreadsheets/d/1rNQuOmfj3YWESN6ryAYsek9ukh8Jth9D/edit?usp=sharing&ouid=108845528456198781082&rtpof=true&sd=true)를 반드시 참고해주세요.

**Recall@5**

```import pandas as pd
import numpy as np


def recall5(answer_df, submission_df):
    """
    Calculate recall@5 for given dataframes.
    
    Parameters:
    - answer_df: DataFrame containing the ground truth
    - submission_df: DataFrame containing the predictions
    
    Returns:
    - recall: Recall@5 value
    """
    
    primary_col = answer_df.columns[0]
    secondary_col = answer_df.columns[1]
    
    # Check if each primary_col entry has exactly 5 secondary_col predictions
    prediction_counts = submission_df.groupby(primary_col).size()
    if not all(prediction_counts == 5):
        raise ValueError(f"Each {primary_col} should have exactly 5 {secondary_col} predictions.")


    # Check for NULL values in the predicted secondary_col
    if submission_df[secondary_col].isnull().any():
        raise ValueError(f"Predicted {secondary_col} contains NULL values.")
    
    # Check for duplicates in the predicted secondary_col for each primary_col
    duplicated_preds = submission_df.groupby(primary_col).apply(lambda x: x[secondary_col].duplicated().any())
    if duplicated_preds.any():
        raise ValueError(f"Predicted {secondary_col} contains duplicates for some {primary_col}.")


    # Filter the submission dataframe based on the primary_col present in the answer dataframe
    submission_df = submission_df[submission_df[primary_col].isin(answer_df[primary_col])]
    
    # For each primary_col, get the top 5 predicted secondary_col values
    top_5_preds = submission_df.groupby(primary_col).apply(lambda x: x[secondary_col].head(5).tolist()).to_dict()
    
    # Convert the answer_df to a dictionary for easier lookup
    true_dict = answer_df.groupby(primary_col).apply(lambda x: x[secondary_col].tolist()).to_dict()
    
    
    individual_recalls = []
    for key, val in true_dict.items():
        if key in top_5_preds:
            correct_matches = len(set(true_dict[key]) & set(top_5_preds[key]))
            individual_recall = correct_matches / min(len(val), 5) # 공정한 평가를 가능하게 위하여 분모(k)를 'min(len(val), 5)' 로 설정함 
            individual_recalls.append(individual_recall)


    recall = np.mean(individual_recalls)
    
    return recall```
