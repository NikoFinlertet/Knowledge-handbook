# 👥 Менторство и развитие команды

## 📋 Навигация
- [Основы менторства](#основы-менторства)
- [Развитие команды](#развитие-команды)
- [Эмоциональный интеллект](#эмоциональный-интеллект)
- [Обучение и коучинг](#обучение-и-коучинг)
- [Управление производительностью](#управление-производительностью)

---

## 🎯 Основы менторства

### Ментор vs Коуч vs Менеджер

```python
class MentorshipRoles:
    def __init__(self):
        self.roles = {
            'mentor': {
                'focus': 'Развитие навыков и карьеры',
                'approach': 'Advice-giving, knowledge sharing',
                'relationship': 'Senior-junior, experience-based',
                'timeline': 'Long-term (6+ месяцев)',
                'activities': [
                    'Career guidance',
                    'Skill development',
                    'Industry insights',
                    'Network building',
                    'Technical expertise sharing'
                ]
            },
            'coach': {
                'focus': 'Unlocking potential',
                'approach': 'Question-asking, self-discovery',
                'relationship': 'Facilitator-learner',
                'timeline': 'Medium-term (3-6 месяцев)',
                'activities': [
                    'Goal setting',
                    'Performance improvement',
                    'Skill enhancement',
                    'Behavioral change',
                    'Problem-solving facilitation'
                ]
            },
            'manager': {
                'focus': 'Performance и results',
                'approach': 'Direction-setting, evaluation',
                'relationship': 'Authority-subordinate',
                'timeline': 'Ongoing',
                'activities': [
                    'Task assignment',
                    'Performance evaluation',
                    'Resource allocation',
                    'Accountability',
                    'Decision making'
                ]
            }
        }
    
    def determine_appropriate_role(self, situation):
        """Определение подходящей роли для ситуации"""
        if situation.needs_guidance:
            return 'mentor'
        elif situation.needs_skill_development:
            return 'coach'
        elif situation.needs_direction:
            return 'manager'
        else:
            return 'assess_further'
```

### Mentorship Framework

```python
class MentorshipFramework:
    def __init__(self):
        self.stages = {
            'initiation': {
                'duration': '2-4 недели',
                'goals': [
                    'Установление отношений',
                    'Понимание потребностей',
                    'Определение целей',
                    'Создание плана развития'
                ],
                'activities': [
                    'Getting to know sessions',
                    'Skill assessment',
                    'Goal setting workshops',
                    'Expectation alignment'
                ]
            },
            'development': {
                'duration': '3-6 месяцев',
                'goals': [
                    'Skill building',
                    'Knowledge transfer',
                    'Project experience',
                    'Network expansion'
                ],
                'activities': [
                    'Regular 1-on-1 meetings',
                    'Code reviews',
                    'Project collaboration',
                    'Knowledge sharing sessions'
                ]
            },
            'mastery': {
                'duration': '3+ месяцев',
                'goals': [
                    'Independent execution',
                    'Leadership opportunities',
                    'Mentoring others',
                    'Career advancement'
                ],
                'activities': [
                    'Leading initiatives',
                    'Peer mentoring',
                    'Conference presentations',
                    'Career planning'
                ]
            }
        }
    
    def create_mentorship_plan(self, mentee_profile):
        """Создание плана менторства"""
        plan = {
            'mentee_assessment': {
                'current_level': self.assess_current_level(mentee_profile),
                'learning_style': self.identify_learning_style(mentee_profile),
                'career_goals': mentee_profile.career_goals,
                'skill_gaps': self.identify_skill_gaps(mentee_profile)
            },
            'development_goals': self.set_smart_goals(mentee_profile),
            'learning_activities': self.design_learning_activities(mentee_profile),
            'milestones': self.define_milestones(),
            'success_metrics': self.define_success_metrics()
        }
        
        return plan
    
    def conduct_mentoring_session(self, session_type):
        """Проведение сессии менторства"""
        session_structures = {
            'skill_development': {
                'structure': [
                    'Current challenge discussion (10 min)',
                    'Concept explanation (20 min)',
                    'Hands-on practice (20 min)',
                    'Q&A and next steps (10 min)'
                ],
                'tools': ['Code review', 'Pair programming', 'Whiteboarding']
            },
            'career_guidance': {
                'structure': [
                    'Career goals review (15 min)',
                    'Market insights sharing (20 min)',
                    'Action planning (20 min)',
                    'Networking opportunities (5 min)'
                ],
                'tools': ['Career matrix', 'Industry trends', 'Role models']
            },
            'project_retrospective': {
                'structure': [
                    'Project review (15 min)',
                    'Lessons learned (20 min)',
                    'Skill application (15 min)',
                    'Future opportunities (10 min)'
                ],
                'tools': ['Retrospective canvas', 'Skill mapping', 'Action items']
            }
        }
        
        return session_structures.get(session_type, {})
```

---

## 🚀 Развитие команды

### Team Development Stages

```python
class TeamDevelopmentFramework:
    def __init__(self):
        self.tuckman_stages = {
            'forming': {
                'characteristics': [
                    'Politeness and courtesy',
                    'Uncertainty about roles',
                    'Dependence on leader',
                    'Low productivity'
                ],
                'leader_actions': [
                    'Provide clear direction',
                    'Establish ground rules',
                    'Build relationships',
                    'Set expectations'
                ],
                'duration': '2-4 недели'
            },
            'storming': {
                'characteristics': [
                    'Conflicts and disagreements',
                    'Resistance to authority',
                    'Subgroup formation',
                    'Emotional reactions'
                ],
                'leader_actions': [
                    'Facilitate conflict resolution',
                    'Encourage open communication',
                    'Mediate disputes',
                    'Maintain focus on goals'
                ],
                'duration': '4-8 недель'
            },
            'norming': {
                'characteristics': [
                    'Emerging cooperation',
                    'Established norms',
                    'Role clarity',
                    'Increased trust'
                ],
                'leader_actions': [
                    'Support norm development',
                    'Encourage collaboration',
                    'Delegate responsibilities',
                    'Facilitate team bonding'
                ],
                'duration': '3-6 месяцев'
            },
            'performing': {
                'characteristics': [
                    'High performance',
                    'Self-organization',
                    'Effective processes',
                    'Strong relationships'
                ],
                'leader_actions': [
                    'Provide strategic direction',
                    'Focus on continuous improvement',
                    'Challenge the team',
                    'Celebrate achievements'
                ],
                'duration': 'Ongoing'
            }
        }
    
    def assess_team_stage(self, team_observations):
        """Оценка стадии развития команды"""
        indicators = {
            'communication_patterns': team_observations.communication,
            'conflict_frequency': team_observations.conflicts,
            'decision_making_speed': team_observations.decisions,
            'productivity_level': team_observations.productivity,
            'member_satisfaction': team_observations.satisfaction
        }
        
        stage_scores = {}
        for stage, details in self.tuckman_stages.items():
            score = self.calculate_stage_alignment(indicators, details)
            stage_scores[stage] = score
        
        return max(stage_scores.items(), key=lambda x: x[1])[0]
    
    def create_development_plan(self, current_stage, target_stage):
        """Создание плана развития команды"""
        return {
            'current_stage': current_stage,
            'target_stage': target_stage,
            'development_activities': self.get_stage_activities(current_stage),
            'timeline': self.estimate_transition_time(current_stage, target_stage),
            'success_indicators': self.define_stage_success_indicators(target_stage)
        }
```

### Skill Development Matrix

```python
class SkillDevelopmentMatrix:
    def __init__(self):
        self.skill_categories = {
            'technical_skills': {
                'programming_languages': ['JavaScript', 'Python', 'Java', 'Go'],
                'frameworks': ['React', 'Node.js', 'Django', 'Spring'],
                'databases': ['PostgreSQL', 'MongoDB', 'Redis'],
                'cloud_platforms': ['AWS', 'Azure', 'GCP'],
                'devops_tools': ['Docker', 'Kubernetes', 'CI/CD']
            },
            'soft_skills': {
                'communication': ['Written', 'Verbal', 'Presentation'],
                'collaboration': ['Teamwork', 'Conflict resolution', 'Feedback'],
                'leadership': ['Mentoring', 'Decision making', 'Vision'],
                'problem_solving': ['Analysis', 'Creativity', 'Critical thinking']
            },
            'domain_knowledge': {
                'business_understanding': ['Domain expertise', 'User needs'],
                'architecture': ['System design', 'Patterns', 'Trade-offs'],
                'process_knowledge': ['SDLC', 'Agile', 'DevOps']
            }
        }
        
        self.proficiency_levels = {
            'novice': {
                'description': 'Beginner, needs guidance',
                'characteristics': ['Learning basics', 'Requires supervision'],
                'development_time': '0-6 months'
            },
            'competent': {
                'description': 'Can work independently',
                'characteristics': ['Solid foundation', 'Minimal supervision'],
                'development_time': '6-18 months'
            },
            'proficient': {
                'description': 'High competency, can teach others',
                'characteristics': ['Expert level', 'Can mentor others'],
                'development_time': '18+ months'
            },
            'expert': {
                'description': 'Industry recognized expertise',
                'characteristics': ['Thought leader', 'Innovation driver'],
                'development_time': '3+ years'
            }
        }
    
    def assess_team_skills(self, team_members):
        """Оценка навыков команды"""
        team_matrix = {}
        
        for member in team_members:
            member_skills = {}
            for category, skills in self.skill_categories.items():
                member_skills[category] = {}
                for skill_area, skill_list in skills.items():
                    member_skills[category][skill_area] = {
                        skill: self.assess_individual_skill(member, skill)
                        for skill in skill_list
                    }
            team_matrix[member.name] = member_skills
        
        return team_matrix
    
    def identify_skill_gaps(self, team_matrix, project_requirements):
        """Выявление пробелов в навыках"""
        gaps = {}
        
        for requirement in project_requirements:
            required_skills = requirement.skills
            available_skills = self.get_team_skills(team_matrix)
            
            for skill in required_skills:
                if skill not in available_skills or available_skills[skill] < requirement.proficiency_level:
                    gaps[skill] = {
                        'required_level': requirement.proficiency_level,
                        'current_level': available_skills.get(skill, 'novice'),
                        'gap_size': self.calculate_gap_size(
                            available_skills.get(skill, 'novice'),
                            requirement.proficiency_level
                        )
                    }
        
        return gaps
    
    def create_skill_development_plan(self, skill_gaps, team_matrix):
        """Создание плана развития навыков"""
        development_plan = {}
        
        for skill, gap_info in skill_gaps.items():
            candidates = self.identify_development_candidates(skill, team_matrix)
            
            development_plan[skill] = {
                'priority': self.calculate_skill_priority(skill, gap_info),
                'development_candidates': candidates,
                'development_approaches': self.suggest_development_approaches(skill),
                'timeline': self.estimate_development_time(gap_info['gap_size']),
                'success_metrics': self.define_skill_success_metrics(skill)
            }
        
        return development_plan
```

---

## 🧠 Эмоциональный интеллект

### EQ Framework для лидеров

```python
class EmotionalIntelligenceFramework:
    def __init__(self):
        self.eq_components = {
            'self_awareness': {
                'description': 'Понимание своих эмоций и их влияния',
                'skills': [
                    'Emotional awareness',
                    'Accurate self-assessment',
                    'Self-confidence'
                ],
                'development_activities': [
                    'Mindfulness meditation',
                    '360-degree feedback',
                    'Emotional journaling',
                    'Self-reflection practices'
                ],
                'assessment_questions': [
                    'Как я реагирую на стресс?',
                    'Какие ситуации вызывают у меня сильные эмоции?',
                    'Как мои эмоции влияют на команду?'
                ]
            },
            'self_regulation': {
                'description': 'Управление своими эмоциями и поведением',
                'skills': [
                    'Emotional self-control',
                    'Adaptability',
                    'Achievement orientation',
                    'Positive outlook'
                ],
                'development_activities': [
                    'Stress management techniques',
                    'Pause and reflect practices',
                    'Cognitive reframing',
                    'Goal setting and tracking'
                ],
                'assessment_questions': [
                    'Как я справляюсь с давлением?',
                    'Могу ли я сохранять спокойствие в конфликтах?',
                    'Как быстро я адаптируюсь к изменениям?'
                ]
            },
            'social_awareness': {
                'description': 'Понимание эмоций других людей',
                'skills': [
                    'Empathy',
                    'Organizational awareness',
                    'Service orientation'
                ],
                'development_activities': [
                    'Active listening practice',
                    'Perspective-taking exercises',
                    'Cultural sensitivity training',
                    'Observation skill development'
                ],
                'assessment_questions': [
                    'Замечаю ли я изменения в настроении команды?',
                    'Понимаю ли я невысказанные потребности коллег?',
                    'Чувствую ли я динамику группы?'
                ]
            },
            'relationship_management': {
                'description': 'Эффективное взаимодействие с другими',
                'skills': [
                    'Influence',
                    'Coach and mentor',
                    'Conflict management',
                    'Team leadership',
                    'Inspirational leadership'
                ],
                'development_activities': [
                    'Communication skills training',
                    'Negotiation practice',
                    'Feedback delivery workshops',
                    'Team building exercises'
                ],
                'assessment_questions': [
                    'Могу ли я мотивировать других?',
                    'Эффективно ли я разрешаю конфликты?',
                    'Создаю ли я позитивную рабочую атмосферу?'
                ]
            }
        }
    
    def assess_emotional_intelligence(self, individual):
        """Оценка эмоционального интеллекта"""
        assessment_results = {}
        
        for component, details in self.eq_components.items():
            scores = []
            for skill in details['skills']:
                skill_score = self.evaluate_eq_skill(individual, skill)
                scores.append(skill_score)
            
            component_score = sum(scores) / len(scores)
            assessment_results[component] = {
                'score': component_score,
                'skill_scores': dict(zip(details['skills'], scores)),
                'development_priority': self.calculate_development_priority(component_score),
                'suggested_activities': details['development_activities']
            }
        
        return assessment_results
    
    def create_eq_development_plan(self, assessment_results):
        """Создание плана развития EQ"""
        plan = {
            'focus_areas': [],
            'development_activities': [],
            'timeline': '6-12 месяцев',
            'success_metrics': []
        }
        
        # Identify top 2 focus areas
        sorted_components = sorted(
            assessment_results.items(),
            key=lambda x: x[1]['score']
        )
        
        for component, results in sorted_components[:2]:
            plan['focus_areas'].append(component)
            plan['development_activities'].extend(results['suggested_activities'][:2])
            plan['success_metrics'].extend(
                self.define_eq_success_metrics(component)
            )
        
        return plan
```

### Team Emotional Dynamics

```python
class TeamEmotionalDynamics:
    def __init__(self):
        self.emotional_states = {
            'high_energy_positive': {
                'characteristics': [
                    'Enthusiasm and excitement',
                    'Optimism about future',
                    'High engagement levels',
                    'Collaborative spirit'
                ],
                'leadership_approach': [
                    'Channel energy toward goals',
                    'Provide challenging work',
                    'Celebrate achievements',
                    'Monitor for burnout'
                ],
                'risks': ['Overcommitment', 'Unrealistic expectations', 'Burnout']
            },
            'low_energy_positive': {
                'characteristics': [
                    'Calm and stable mood',
                    'Satisfaction with status quo',
                    'Steady performance',
                    'Comfortable relationships'
                ],
                'leadership_approach': [
                    'Introduce gentle challenges',
                    'Encourage innovation',
                    'Provide growth opportunities',
                    'Maintain stability'
                ],
                'risks': ['Complacency', 'Stagnation', 'Lack of innovation']
            },
            'high_energy_negative': {
                'characteristics': [
                    'Stress and anxiety',
                    'Frustration with processes',
                    'Conflict and tension',
                    'Reactive behavior'
                ],
                'leadership_approach': [
                    'Address root causes',
                    'Provide emotional support',
                    'Facilitate conflict resolution',
                    'Reduce stressors'
                ],
                'risks': ['Team fragmentation', 'High turnover', 'Poor performance']
            },
            'low_energy_negative': {
                'characteristics': [
                    'Apathy and disengagement',
                    'Resignation to problems',
                    'Low motivation',
                    'Withdrawn behavior'
                ],
                'leadership_approach': [
                    'Rebuild trust and confidence',
                    'Provide clear vision',
                    'Create small wins',
                    'Individual attention'
                ],
                'risks': ['Team dissolution', 'Talent loss', 'Project failure']
            }
        }
    
    def assess_team_emotional_state(self, team_indicators):
        """Оценка эмоционального состояния команды"""
        energy_level = self.measure_energy_level(team_indicators)
        emotional_valence = self.measure_emotional_valence(team_indicators)
        
        if energy_level > 0.6 and emotional_valence > 0.6:
            return 'high_energy_positive'
        elif energy_level <= 0.6 and emotional_valence > 0.6:
            return 'low_energy_positive'
        elif energy_level > 0.6 and emotional_valence <= 0.6:
            return 'high_energy_negative'
        else:
            return 'low_energy_negative'
    
    def develop_response_strategy(self, emotional_state):
        """Разработка стратегии реагирования на эмоциональное состояние"""
        state_info = self.emotional_states[emotional_state]
        
        return {
            'immediate_actions': state_info['leadership_approach'][:2],
            'medium_term_strategy': state_info['leadership_approach'][2:],
            'risk_mitigation': self.create_risk_mitigation_plan(state_info['risks']),
            'success_indicators': self.define_emotional_success_indicators(emotional_state)
        }
```

---

## 📚 Обучение и коучинг

### Learning & Development Framework

```python
class LearningDevelopmentFramework:
    def __init__(self):
        self.learning_styles = {
            'visual': {
                'characteristics': ['Learns through seeing', 'Prefers diagrams'],
                'methods': ['Diagrams', 'Charts', 'Presentations', 'Videos']
            },
            'auditory': {
                'characteristics': ['Learns through hearing', 'Enjoys discussions'],
                'methods': ['Lectures', 'Discussions', 'Podcasts', 'Verbal explanations']
            },
            'kinesthetic': {
                'characteristics': ['Learns through doing', 'Hands-on approach'],
                'methods': ['Practice', 'Experiments', 'Simulations', 'Real projects']
            },
            'reading_writing': {
                'characteristics': ['Learns through text', 'Prefers written info'],
                'methods': ['Documentation', 'Tutorials', 'Code examples', 'Written exercises']
            }
        }
    
    def design_learning_program(self, learning_objectives, learner_profile):
        """Проектирование программы обучения"""
        program = {
            'learning_objectives': learning_objectives,
            'learner_assessment': {
                'current_level': learner_profile.skill_level,
                'learning_style': learner_profile.preferred_learning_style,
                'motivation_factors': learner_profile.motivation,
                'time_availability': learner_profile.available_time
            },
            'learning_path': self.create_learning_path(learning_objectives),
            'delivery_methods': self.select_delivery_methods(learner_profile),
            'assessment_strategy': self.design_assessment_strategy(learning_objectives),
            'support_resources': self.identify_support_resources()
        }
        
        return program
    
    def create_learning_path(self, objectives):
        """Создание траектории обучения"""
        path_stages = {
            'foundation': {
                'duration': '2-4 недели',
                'focus': 'Basic concepts and terminology',
                'activities': ['Reading', 'Videos', 'Basic exercises'],
                'assessment': 'Knowledge check quizzes'
            },
            'application': {
                'duration': '4-8 недель',
                'focus': 'Practical application of concepts',
                'activities': ['Hands-on projects', 'Code reviews', 'Pair programming'],
                'assessment': 'Project deliverables'
            },
            'mastery': {
                'duration': '4-12 недель',
                'focus': 'Advanced concepts and best practices',
                'activities': ['Complex projects', 'Teaching others', 'Innovation'],
                'assessment': 'Peer review and presentation'
            }
        }
        
        return path_stages
```

### Coaching Techniques

```python
class CoachingTechniques:
    def __init__(self):
        self.coaching_models = {
            'grow': {
                'goal': 'What do you want to achieve?',
                'reality': 'What is the current situation?',
                'options': 'What could you do?',
                'will': 'What will you do?'
            },
            'solution_focused': {
                'preferred_future': 'What would success look like?',
                'scaling': 'On a scale of 1-10, where are you now?',
                'exception_finding': 'When has this worked before?',
                'next_steps': 'What\'s the smallest step forward?'
            },
            'appreciative_inquiry': {
                'discover': 'What\'s working well?',
                'dream': 'What could be possible?',
                'design': 'What should be?',
                'deploy': 'What will be?'
            }
        }
    
    def conduct_coaching_session(self, coachee, session_goal):
        """Проведение коучинговой сессии"""
        session_structure = {
            'opening': {
                'duration': '5 minutes',
                'activities': [
                    'Check-in and rapport building',
                    'Review previous action items',
                    'Set session agenda'
                ]
            },
            'exploration': {
                'duration': '20 minutes',
                'activities': [
                    'Explore current situation',
                    'Identify challenges and opportunities',
                    'Clarify goals and objectives'
                ]
            },
            'options_generation': {
                'duration': '20 minutes',
                'activities': [
                    'Brainstorm possible solutions',
                    'Evaluate options and trade-offs',
                    'Identify resources and support'
                ]
            },
            'commitment': {
                'duration': '10 minutes',
                'activities': [
                    'Agree on specific actions',
                    'Set timelines and accountability',
                    'Plan follow-up'
                ]
            },
            'closing': {
                'duration': '5 minutes',
                'activities': [
                    'Summarize key insights',
                    'Confirm next steps',
                    'Schedule follow-up'
                ]
            }
        }
        
        return session_structure
```

---

## 📊 Управление производительностью

### Performance Management System

```python
class PerformanceManagementSystem:
    def __init__(self):
        self.performance_dimensions = {
            'technical_excellence': {
                'weight': 0.4,
                'indicators': [
                    'Code quality and best practices',
                    'Problem-solving ability',
                    'Technical innovation',
                    'System design skills'
                ]
            },
            'delivery_execution': {
                'weight': 0.3,
                'indicators': [
                    'Meeting deadlines and commitments',
                    'Quality of deliverables',
                    'Efficiency and productivity',
                    'Handling of complex projects'
                ]
            },
            'collaboration_leadership': {
                'weight': 0.2,
                'indicators': [
                    'Teamwork and cooperation',
                    'Communication effectiveness',
                    'Mentoring and knowledge sharing',
                    'Conflict resolution'
                ]
            },
            'growth_learning': {
                'weight': 0.1,
                'indicators': [
                    'Continuous learning',
                    'Adaptability to change',
                    'Seeking feedback',
                    'Professional development'
                ]
            }
        }
    
    def conduct_performance_review(self, employee, review_period):
        """Проведение оценки производительности"""
        review_process = {
            'preparation': {
                'self_assessment': 'Employee completes self-evaluation',
                'goal_review': 'Review progress against set goals',
                'feedback_collection': 'Gather 360-degree feedback',
                'data_analysis': 'Analyze performance metrics'
            },
            'evaluation': {
                'dimension_scoring': self.score_performance_dimensions(employee),
                'goal_achievement': self.assess_goal_achievement(employee),
                'behavioral_assessment': self.evaluate_behaviors(employee),
                'overall_rating': self.calculate_overall_rating(employee)
            },
            'discussion': {
                'strengths_recognition': 'Acknowledge strong performance',
                'improvement_areas': 'Discuss development opportunities',
                'career_conversation': 'Explore career aspirations',
                'support_planning': 'Plan support and resources'
            },
            'planning': {
                'goal_setting': 'Set goals for next period',
                'development_plan': 'Create individual development plan',
                'success_metrics': 'Define success indicators',
                'follow_up_schedule': 'Plan regular check-ins'
            }
        }
        
        return review_process
    
    def create_improvement_plan(self, performance_gaps):
        """Создание плана улучшения производительности"""
        plan = {
            'performance_gaps': performance_gaps,
            'root_cause_analysis': self.analyze_performance_issues(performance_gaps),
            'improvement_strategies': self.develop_improvement_strategies(performance_gaps),
            'support_resources': self.identify_support_resources(performance_gaps),
            'timeline': self.create_improvement_timeline(),
            'success_metrics': self.define_improvement_metrics(performance_gaps),
            'monitoring_process': self.design_monitoring_process()
        }
        
        return plan
```

---

## 📈 Метрики развития команды

### Team Development Metrics

```python
class TeamDevelopmentMetrics:
    def __init__(self):
        self.metric_categories = {
            'team_performance': [
                'Velocity trends',
                'Quality metrics (defect rates)',
                'Cycle time',
                'Deployment frequency',
                'Team satisfaction scores'
            ],
            'individual_growth': [
                'Skill development progress',
                'Career advancement',
                'Learning goals achievement',
                'Mentoring relationships',
                'Knowledge sharing contributions'
            ],
            'team_dynamics': [
                'Psychological safety index',
                'Communication effectiveness',
                'Conflict resolution time',
                'Decision-making speed',
                'Trust levels'
            ],
            'leadership_effectiveness': [
                'Team retention rates',
                'Leadership pipeline strength',
                '360-degree feedback scores',
                'Team engagement levels',
                'Innovation metrics'
            ]
        }
    
    def measure_team_health(self, team):
        """Измерение здоровья команды"""
        health_indicators = {}
        
        for category, metrics in self.metric_categories.items():
            category_scores = []
            for metric in metrics:
                score = self.collect_metric_data(team, metric)
                category_scores.append(score)
            
            health_indicators[category] = {
                'average_score': sum(category_scores) / len(category_scores),
                'individual_scores': dict(zip(metrics, category_scores)),
                'trend': self.calculate_trend(category, team),
                'action_items': self.generate_action_items(category, category_scores)
            }
        
        return health_indicators
```

---

## 🔗 Связанные разделы

- [[technical-leadership|Technical Leadership]] - Техническое лидерство
- [[influence-decision-making|Influence & Decisions]] - Влияние и решения  
- [[engineering-culture|Engineering Culture]] - Инженерная культура
- [[conflict-management|Conflict Management]] - Управление конфликтами
- [[../team-management|Team Management]] - Управление командой 