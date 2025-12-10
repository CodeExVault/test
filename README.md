'use client'

// Activity Tracking Hook
// Automatically tracks user activities for streaks and weekly challenges

import { useCallback } from 'react'
import { ACTIVITY_TYPES, getChallengTypeForActivity, type ActivityType } from './engagement-utils'
import { trackEngagementEvent } from './analytics-client'

export function useActivityTracking() {
  /**
   * Track an activity and update streaks/challenges
   * Call this after any significant user action
   */
  const trackActivity = useCallback(async (activityType: ActivityType, xpEarned?: number) => {
    try {
      // Update streak
      await fetch('/api/streaks/update', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          activityTypes: [activityType],
          xpEarned: xpEarned ?? 0
        })
      })
      trackEngagementEvent('activity_tracked', { activityType })
      
      // Update relevant challenges
      const challengeType = getChallengTypeForActivity(activityType)
      if (challengeType) {
        await fetch('/api/challenges/progress', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            challengeType,
            increment: 1
          })
        })
        trackEngagementEvent('challenge_progress_updated', { activityType, challengeType })
      }
    } catch (error) {
      console.error('Error tracking activity:', error)
      // Fail silently - don't interrupt user experience
    }
  }, [])
  
  /**
   * Track lab completion
   */
  const trackLabCompletion = useCallback(async () => {
    trackEngagementEvent('lab_completed')
    await trackActivity(ACTIVITY_TYPES.LAB_COMPLETED)
  }, [trackActivity])
  
  /**
   * Track lesson completion
   */
  const trackLessonCompletion = useCallback(async () => {
    trackEngagementEvent('lesson_completed')
    await trackActivity(ACTIVITY_TYPES.LESSON_COMPLETED)
  }, [trackActivity])
  
  /**
   * Track course completion
   */
  const trackCourseCompletion = useCallback(async () => {
    trackEngagementEvent('course_completed')
    await trackActivity(ACTIVITY_TYPES.COURSE_COMPLETED)
  }, [trackActivity])
  
  /**
   * Track community reply
   */
  const trackCommunityReply = useCallback(async () => {
    trackEngagementEvent('community_reply_created')
    await trackActivity(ACTIVITY_TYPES.COMMUNITY_REPLY)
  }, [trackActivity])
  
  /**
   * Track helpful mark given
   */
  const trackHelpfulMark = useCallback(async () => {
    trackEngagementEvent('helpful_mark_received')
    await trackActivity(ACTIVITY_TYPES.HELPFUL_MARK)
  }, [trackActivity])
  
  /**
   * Track answer mark given
   */
  const trackAnswerMark = useCallback(async () => {
    trackEngagementEvent('answer_mark_received')
    await trackActivity(ACTIVITY_TYPES.ANSWER_MARK)
  }, [trackActivity])
  
  /**
   * Track daily login (call once per day on app load)
   */
  const trackDailyLogin = useCallback(async () => {
    // Check if we've already tracked today
    const lastTracked = localStorage.getItem('lastLoginTracked')
    const today = new Date().toDateString()
    
    if (lastTracked === today) {
      return // Already tracked today
    }
    
    trackEngagementEvent('daily_login_tracked')
    await trackActivity(ACTIVITY_TYPES.DAILY_LOGIN)
    localStorage.setItem('lastLoginTracked', today)
  }, [trackActivity])
  
  return {
    trackActivity,
    trackLabCompletion,
    trackLessonCompletion,
    trackCourseCompletion,
    trackCommunityReply,
    trackHelpfulMark,
    trackAnswerMark,
    trackDailyLogin,
  }
}

