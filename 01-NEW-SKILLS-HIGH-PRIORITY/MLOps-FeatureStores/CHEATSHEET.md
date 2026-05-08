# MLOps & Feature Stores Cheatsheet - Quick Reference

## MLOps Fundamentals
- **MLOps**: DevOps for machine learning (automate ML lifecycle)
- **ML Lifecycle**: Data → Train → Validate → Deploy → Monitor → Retrain
- **Experiment tracking**: Log parameters, metrics, artifacts (MLflow, Weights & Biases)
- **Model registry**: Centralized model versioning and metadata
- **Model versioning**: Track model versions (staging, production)
- **Reproducibility**: Same code/data/config → same results

## Feature Store
- **Feature Store**: Centralized repository for ML features
- **Features**: Transformed input variables for ML models
- **Feature engineering**: Create features from raw data
- **Online features**: Low-latency serving for real-time predictions
- **Offline features**: Batch features for training
- **Point-in-time correctness**: Features as of prediction time (no data leakage)
- **Feature discovery**: Search and reuse existing features

## Model Training
- **Training pipeline**: Automated workflow (data → preprocess → train → validate)
- **Hyperparameter tuning**: Optimize model parameters (Optuna, Ray Tune, Grid/Random search)
- **Cross-validation**: Split data for unbiased evaluation (k-fold)
- **Train/validation/test split**: 70/15/15 or 80/10/10
- **Overfitting**: Model memorizes training data (regularization, dropout)
- **Underfitting**: Model too simple (add features, increase complexity)

## Model Deployment
- **Batch inference**: Predictions on large dataset (offline)
- **Real-time inference**: Low-latency predictions via API
- **Model serving**: Expose model as REST API (FastAPI, TensorFlow Serving, Seldon)
- **A/B testing**: Compare models in production (50/50 traffic split)
- **Canary deployment**: Gradual rollout (1% → 10% → 100%)
- **Blue-green deployment**: Two environments, instant switch
- **Shadow mode**: Run new model alongside old (no user impact)

## Model Monitoring
- **Model drift**: Model performance degrades over time
- **Data drift**: Input distribution changes (feature drift)
- **Concept drift**: Relationship between features and target changes
- **Performance monitoring**: Track accuracy, precision, recall, AUC
- **Latency monitoring**: Prediction response time
- **Prediction drift**: Output distribution changes

## CI/CD for ML
- **Continuous Integration**: Automated testing (unit tests, data validation)
- **Continuous Delivery**: Automated model deployment
- **Automated retraining**: Trigger training on data/performance changes
- **Model validation gates**: Metrics thresholds before promotion
- **Rollback strategy**: Revert to previous model if issues

## ML Frameworks
- **MLflow**: Experiment tracking, model registry, deployment
- **Kubeflow**: Kubernetes-native ML platform
- **Airflow**: Orchestrate ML pipelines
- **DVC**: Data version control (like Git for data)
- **Feast / Tecton**: Feature stores
- **Databricks**: Unified analytics + ML platform

## Model Optimization
- **Model compression**: Reduce size (pruning, knowledge distillation)
- **Quantization**: Reduce precision (FP32 → INT8)
- **ONNX**: Open format for model interoperability
- **Model ensembles**: Combine multiple models
- **Caching**: Cache predictions for repeated inputs

## Production Challenges
- **Scalability**: Handle high traffic (horizontal scaling, load balancing)
- **Latency**: Reduce prediction time (model optimization, caching)
- **Cost**: Optimize compute (spot instances, auto-scaling)
- **Security**: Protect model from adversarial attacks
- **Explainability**: SHAP, LIME for model interpretability
- **Bias detection**: Monitor for fairness issues

## Feature Store Tools
- **Feast**: Open-source, cloud-agnostic feature store
- **Tecton**: Managed feature platform (Spark/Flink)
- **Databricks Feature Store**: Integrated with Databricks
- **AWS SageMaker Feature Store**: AWS native
- **Vertex AI Feature Store**: GCP native

## Best Practices
✅ Version everything (data, code, models, configs) | ✅ Automate pipelines | ✅ Monitor in production | ✅ Use feature stores | ✅ Track experiments | ✅ Validate before deployment | ✅ A/B test new models | ✅ Plan for rollback | ✅ Document model decisions | ✅ Implement CI/CD
