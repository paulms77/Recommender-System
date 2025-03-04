# 제1회 국민대학교 AI빅데이터 분석 경진대회 - Dataset

### Dataset Info.
**apply_train.csv [파일]**
- 이력서가 채용 공고에 실제 지원한 관계 목록 (히스토리)
  
**이력서 관련 데이터 [파일]**
- resume.csv
- resume_certificate.csv
- resume_education.csv
- resume_language.csv
  
**채용공고 관련 데이터 [파일]**
- recruitment.csv
- company.csv
  
**sample_submission.csv [파일] - 제출 양식**
- resume_seq : 추천을 진행할 이력서 고유 번호
- recruitment_seq : 이력서에 대해 추천한 채용 공고 고유 번호
- resume.csv에 존재하는 모든 resume_seq에 대해서 5개의 채용 공고를 추천해야 합니다.
- 해당 이력서에서 실제 지원이 이루어졌던 채용 공고는 추천하지 않습니다.
- 중복된 채용 공고를 추천하거나, 5개가 아닌 개수의 채용 공고를 추천하는 경우 제출이 불가능합니다.
